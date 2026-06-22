# Broken access control — IDOR to account takeover

> **Target:** PortSwigger Web Security Academy — *User ID controlled by request parameter*
> **Class:** `IDOR` / `Broken Access Control` · **Difficulty:** ★★☆

## Recon

After logging in as `wiener`, the account page loads with the user in the URL:

```http
GET /my-account?id=wiener HTTP/1.1
Cookie: session=…
```

The page shows `wiener`'s API key. The `id` is the *only* thing identifying whose account is shown —
and it's fully attacker-controlled.

## The bug

The server authenticates the **session** but never checks that the session **owns** the requested
`id`. It trusts the parameter to be honest. Classic missing object-level authorization.

## Exploitation

Swap the value:

```http
GET /my-account?id=administrator HTTP/1.1
Cookie: session=…
```

The response returns the **administrator's** API key. No privilege, no second account — just an
incremented/guessed identifier.

> Where IDs are sequential integers (`?id=1031`), the same bug becomes *enumerable* — script the range
> and harvest every record.

## Impact

Horizontal **and** vertical access: any user's data, including the admin's, by changing one value.
Read API keys → full account takeover.

## Fix

Enforce ownership **server-side**, from the session — never from the request:

```python
# the record is chosen by who you ARE, not what you ASK for
account = db.accounts.get(owner_id=session.user_id)
```

For unavoidable object references, check `record.owner == session.user` before returning, and prefer
unguessable IDs (UUIDs) as defence-in-depth.

## Takeaway

The most under-looked bug class on the web: the app knows *who you are* but forgets to check *what
you're allowed to touch*. Always test every identifier with a second, lower-privilege account.
