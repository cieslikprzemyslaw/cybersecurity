# A05: Injection - Labs and Practice

This category currently uses SQL Injection as the practical A05 example.

## Completed SQL Injection labs

- [UNION-based SQL Injection - retrieving data from other tables](sql-injection/labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](sql-injection/labs/02-blind-conditional-responses.md)

## Practice coverage

The completed practice covers:

- identifying user-controlled input that reaches a SQL query,
- recognising visible SQLi evidence in HTTP responses,
- using `UNION SELECT` to prove data exposure in a legal lab,
- using boolean response differences to reason about blind SQLi,
- translating lab evidence into security finding language,
- defining remediation and regression tests.

## Current limits

This A05 folder does not yet include separate practice for NoSQL Injection, OS Command Injection, SSTI, prompt injection, or other interpreter-specific injection classes.

Those topics should be added as separate modules after practical labs or review tasks are completed.
