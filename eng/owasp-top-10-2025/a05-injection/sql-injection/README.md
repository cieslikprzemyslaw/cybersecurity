# SQL Injection

SQL Injection happens when untrusted input is included in a SQL query in a way that allows the input to change the query logic executed by the database engine.

## Mental model

```text
user-controlled input
  -> SQL query string
  -> database / SQL interpreter
  -> changed query behaviour
```

## Files

- [01-overview.md](01-overview.md) - definition, impact, interpreter, root cause, and main remediation.
- [02-types-of-sqli.md](02-types-of-sqli.md) - main SQLi types and evidence patterns.
- [03-in-band-sqli.md](03-in-band-sqli.md) - error-based and UNION-based SQLi.
- [04-union-based-sqli.md](04-union-based-sqli.md) - UNION-specific testing notes and lab evidence flow.
- [05-blind-sqli.md](05-blind-sqli.md) - boolean-based blind SQLi and oracle thinking.
- [06-out-of-band-sqli.md](06-out-of-band-sqli.md) - OOB concepts and constraints.
- [07-filter-bypass-and-encoding.md](07-filter-bypass-and-encoding.md) - encoding and filter-bypass learning notes.
- [08-remediation.md](08-remediation.md) - developer remediation model.
- [cheat-sheet.md](cheat-sheet.md) - practical testing and review reminders.
- [regression-tests.md](regression-tests.md) - SQLi-specific regression checks.
- [learning-summary.md](learning-summary.md) - comparison of completed labs and personal takeaways.

## Completed labs

- [UNION-based SQL Injection - retrieving data from other tables](labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](labs/02-blind-conditional-responses.md)

## Key takeaway

The main fix for SQL Injection is not stronger blacklists or changing GET to POST. The main fix is to use prepared statements / parameterized queries so that SQL structure and user input are handled separately.
