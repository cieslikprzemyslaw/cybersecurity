# In-Band SQL Injection

This file expands the practical side of in-band SQLi. The general type classification is in [02-types-of-sqli.md](02-types-of-sqli.md).

In-band means the evidence comes back through the same HTTP response being tested:

```text
request with controlled input
  -> backend executes SQL query
  -> error, content change, or data returns in the response
```

## What to inspect in the response

The key question is whether the response shows that the input changed SQL query behaviour.

Example signals:

- `500 Internal Server Error` after adding a quote,
- `SQL syntax error`,
- table, column, or database engine details in an error,
- changed result count,
- additional data visible in HTML,
- page behaviour returning to normal after adding a SQL comment.

## Error-based evidence

Error-based evidence is strongest when the response leaks database details or shows that SQL syntax was broken.

In review, the important question is not only "did an error appear?", but:

- does the error depend on controlled input,
- does it look like a database error,
- does it expose implementation details,
- does it disappear when the payload changes in a way that matches the SQLi hypothesis.

## UNION-based evidence

UNION-based evidence is stronger than error confirmation alone because it shows real impact: data from an additional query appears in the normal response.

A good evidence flow:

```text
quote breaks the response
SQL comment restores the response
NULL tests identify column count
text values appear in HTML
data from another table appears in HTML
```

The UNION-specific technique is covered in [04-union-based-sqli.md](04-union-based-sqli.md).

## Why in-band SQLi is important

In-band SQLi is often easier to explain in a report because the evidence is directly visible in the application response.

This helps show:

- controlled input,
- changed backend behaviour,
- user-visible data or errors,
- real impact.

## Secure implementation

- Do not concatenate user input into SQL strings.
- Use prepared statements / parameterized queries.
- Avoid exposing verbose SQL errors.
- Use least-privilege database accounts.
