# SQL Injection Learning Summary

## Completed labs

- [UNION-based SQL Injection - retrieving data from other tables](labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](labs/02-blind-conditional-responses.md)

## UNION-based SQLi lesson

The UNION lab showed how a vulnerable `category` parameter could retrieve data from another table.

Key evidence:

```text
quote broke the query
SQL comment repaired the query
NULL tests confirmed column count
abc/def confirmed visible text columns
users table data appeared in HTML
```

Main lesson:

```text
UNION-based SQLi can expose data directly in the normal web response.
```

## Blind SQLi lesson

The blind lab used the `TrackingId` cookie and a response marker to infer data without seeing it directly.

Key evidence:

```text
Welcome back = TRUE
No Welcome back = FALSE
```

Main lesson:

```text
Blind SQLi uses response behaviour as an oracle for yes/no database questions.
```

## Comparison

| Topic | UNION-based SQLi | Blind SQLi |
|---|---|---|
| Data visible directly? | Yes | No |
| Main evidence | Data appears in response | Response marker changes |
| Example marker | Extracted rows | `Welcome back` |
| Technique | `UNION SELECT` | TRUE/FALSE conditions |
| Impact | Data disclosure/account compromise | Data extraction/account compromise |

## Overall AppSec takeaway

SQL Injection is not only about a payload. It is about proving that user input changes SQL query behaviour and showing the real impact.

The developer fix is prepared statements / parameterized queries, supported by validation, allowlists, safe ORM usage, least privilege, and regression tests.
