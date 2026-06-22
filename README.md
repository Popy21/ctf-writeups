# 🚩 Web Security Write-ups

Walk-throughs from public training platforms ([PortSwigger Web Security Academy](https://portswigger.net/web-security),
[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/), [picoCTF](https://picoctf.org/)) and CTF events I play —
each one source → sink → exploit → fix, the way I report real bugs.

> Method: read the code/flow, find the broken invariant, **prove it with a PoC**. *Look where people don't look.*
> Maintained by [@Popy21](https://github.com/Popy21) · Apple WebKit [CVE-2026-28962](https://support.apple.com/en-us/127121) · 53 reports on [YesWeHack](https://yeswehack.com/hunters/BaguettePwnM)

## Index

| # | Write-up | Class | Difficulty |
|---|----------|-------|------------|
| 01 | [SQL injection — UNION data exfiltration](web/01-sqli-union.md) | `SQLi` | ★★☆ |
| 02 | [Broken access control — IDOR to account takeover](web/02-idor-broken-access-control.md) | `IDOR` / `BAC` | ★★☆ |
| 03 | [SSRF — reaching cloud metadata](web/03-ssrf-cloud-metadata.md) | `SSRF` | ★★★ |
| 04 | [DOM-based XSS — client-side sink](web/04-dom-xss.md) | `DOM-XSS` | ★★☆ |

*New write-up?* Copy [`TEMPLATE.md`](TEMPLATE.md).

## Why these classes

They're the same bug classes I report in bounty programs — `DOM-XSS`, `Open Redirect`, `IDOR`,
`Broken Access Control`, `SSRF`. The labs below are reproducible practice; the methodology is what
carries over to live targets.

> ⚖️ Everything here is performed on **intentionally-vulnerable, public training targets** or in
> sanctioned CTFs. Never against systems without authorization.
