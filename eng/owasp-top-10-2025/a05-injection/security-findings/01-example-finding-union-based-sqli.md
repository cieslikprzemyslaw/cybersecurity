# Example Finding: UNION-Based SQL Injection in Category Filter

## Summary

The product category filter is vulnerable to UNION-based SQL Injection. An attacker can manipulate the `category` parameter to retrieve data from another database table.

## Affected parameter

```text
GET /filter?category=...
```

Affected input:

```text
category
```

## Evidence

The following testing sequence demonstrated the issue:

```text
Normal category value -> 200 OK
Single quote -> 500 Internal Server Error
Single quote + SQL comment -> normal response
UNION SELECT NULL -> error
UNION SELECT NULL,NULL -> valid response
UNION SELECT text values -> injected values appeared in HTML
UNION SELECT username,password FROM users -> user credentials appeared in HTML
```

This proves that the `category` parameter can change the SQL query and that data from another table can be rendered in the response.

## Impact

An attacker can retrieve sensitive information from the database. In the lab scenario, this included usernames and passwords from the `users` table, which allowed administrator account compromise.

## Root cause

The application likely builds a SQL query by concatenating the `category` parameter into the SQL string.

## Recommendation

- Use prepared statements / parameterized queries.
- Do not concatenate user-controlled values into SQL strings.
- Use an allowlist for valid category values.
- Avoid exposing database errors.
- Use least-privilege database accounts.
- Add regression tests for SQLi payloads.

## Regression tests

- A quote in the category value should not cause a database error.
- SQL comments should not change query behaviour.
- UNION SELECT payloads should not return additional rows.
- Attempts to retrieve `users` table data should fail safely.
