# SQLi Filter Bypass and Encoding

## Why this matters

Some applications try to prevent SQL Injection by removing or blocking certain characters or keywords.

Examples:

- quotes,
- spaces,
- `OR`,
- `AND`,
- `UNION`,
- `SELECT`.

This is fragile because filters and SQL parsers may interpret input differently.

## Encoding lesson

If a payload appears in the generated SQL query as literal encoded text, such as `%20`, `%27`, or `%7C`, it may mean the payload was not decoded at the expected layer.

Important distinction:

```text
UI input -> use raw payload if frontend encodes input
Direct URL / Burp -> use encoded payload when needed
```

## No quote scenarios

Some filters remove single or double quotes.

Possible concepts in authorised labs:

- numeric contexts may not need quotes,
- strings may be built using functions such as `CONCAT()` or `CHAR()`,
- hex values may represent strings depending on the database.

The goal is not to memorise payloads. The goal is to understand whether the SQL parser still receives valid syntax.

## No spaces scenarios

If spaces are blocked or removed, alternatives may include:

- SQL comments such as `/**/`,
- tab: `%09`,
- newline: `%0A`,
- form feed: `%0C`,
- carriage return: `%0D`,
- other whitespace variants depending on parser behaviour.

## Blocked keywords

If keywords such as `OR` or `AND` are removed, some databases may support alternatives such as `||` for OR-like logic.

Keyword splitting with comments may also work in some contexts:

```sql
SE/**/LECT
```

This depends on the database and filter behaviour.

## Important AppSec lesson

Blacklists are weak because attackers may find other valid syntax that the SQL parser accepts.

The secure fix is not a stronger blacklist.

The secure fix is:

- prepared statements / parameterized queries,
- safe ORM usage,
- allowlists where values are limited,
- safe error handling,
- regression tests.

## Safe learning note

These techniques should only be used in legal labs or authorised testing environments.
