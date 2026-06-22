# DOM-based XSS — client-side sink

> **Target:** PortSwigger Web Security Academy — *DOM XSS using `innerHTML` sink with `location.search` source*
> **Class:** `DOM-XSS` · **Difficulty:** ★★☆

## Recon

The search box writes the query back into the page **client-side** — no server round-trip. The
JavaScript looks like:

```js
const query = new URLSearchParams(location.search).get('search');
document.getElementById('searchMessage').innerHTML = 'Results for ' + query;
```

**Source:** `location.search` (attacker-controlled). **Sink:** `innerHTML` (renders markup). The data
flows from URL → DOM with no sanitization. That's the whole bug, and it never touches the server — so
a server-side WAF never sees it.

## The bug

Untrusted input reaches a dangerous DOM sink. `innerHTML` parses HTML, so injected markup executes.

## Exploitation

`<script>` won't run when set via `innerHTML`, so use an auto-firing event handler:

```
https://lab.example/?search=<img src=x onerror=alert(document.domain)>
```

The image fails to load → `onerror` fires → JS executes in the victim's origin. Swap `alert()` for a
real payload (session/token exfiltration, action-on-behalf).

> This exact pattern — `location.hash`/`location.search` → `innerHTML` — is what I found live on a PSD2
> login flow during a bounty (reported responsibly). Client-side sinks are where the overlooked bugs
> live, because scanners and WAFs watching the server never see them.

## Impact

Arbitrary JS in the victim's session: cookie/token theft, credential keylogging on the page, or any
authenticated action — full client-side compromise of that origin.

## Fix

- Write text with `textContent` / `innerText`, never `innerHTML`, for untrusted data.
- If HTML is required, sanitize with a vetted library (DOMPurify).
- Add a strict **Content-Security-Policy** as defence-in-depth.

## Takeaway

Read the JavaScript, not just the responses. Map every **source** (`location.*`, `document.referrer`,
`postMessage`) to every **sink** (`innerHTML`, `document.write`, `eval`) — the bug is the path between
them, and it's invisible from the server side.
