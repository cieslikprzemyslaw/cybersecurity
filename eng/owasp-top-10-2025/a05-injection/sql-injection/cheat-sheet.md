# SQL Injection Cheat Sheet

This cheat sheet is for learning and authorised lab testing only.

## Mental model

```text
input -> SQL query -> database interpreter -> changed behaviour
```

## Evidence patterns

```text
Error-based:
Database error appears in the response.

Union-based:
Data from another SELECT query appears in the normal response.

Boolean-based blind:
TRUE and FALSE conditions produce different responses.

Time-based blind:
TRUE condition causes a measurable delay.

Out-of-band:
Evidence appears through DNS/HTTP/SMB callback or external file/request.
```

## Basic test flow

```text
1. Identify controlled input.
2. Send a normal request.
3. Add a quote and observe behaviour.
4. Add a SQL comment and check whether behaviour returns to normal.
5. If this is a UNION lab, identify the column count.
6. Check text-compatible columns.
7. Prove real impact carefully.
```

## UNION reminders

```text
UNION SELECT must return the same number of columns as the original query.
NULL helps test column count.
Text values like 'abc','def' help prove visible text-compatible columns.
```

## Blind SQLi reminders

```text
Blind SQLi is about asking yes/no questions.
Find a reliable marker.
TRUE -> marker appears
FALSE -> marker disappears
```

Example marker from the lab:

```text
Welcome back
```

## Encoding reminders

```text
Raw payload in UI if frontend encodes input.
Encoded payload in URL/Burp when sending directly.
If response shows %20 or %27 literally, payload may not have decoded as expected.
```

## Filter bypass concepts

- `||` may work as an alternative to `OR` depending on the database.
- `%0A`, `%09`, `%0C`, `%0D` may act as whitespace alternatives.
- `/**/` comments may replace spaces.
- `CONCAT()`, `CHAR()`, or hex may build strings without quotes depending on the database.
- Numeric contexts may not need quotes.

## Fix reminders

```text
Main fix: prepared statements / parameterized queries.
Supporting controls: allowlists, validation, safe ORM usage, least privilege, generic errors, logging, regression tests.
```
