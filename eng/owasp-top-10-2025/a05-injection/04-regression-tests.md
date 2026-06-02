# A05: Injection - Regression Test Ideas

Regression tests should prove that previously dangerous input is now treated as data, not executable syntax.

## SQL Injection regression tests

- A single quote in a category or search value should not cause a 500 error.
- SQL comments such as `--` should not change query behaviour.
- Boolean payloads such as `' AND '1'='1` and `' AND '1'='2` should not produce meaningful response differences.
- `UNION SELECT NULL,NULL` should not return additional rows or data.
- `UNION SELECT 'abc','def'` should not render attacker-controlled SQL result data.
- Cookie values such as `TrackingId` should not be usable to alter SQL query logic.
- SQLi payloads should be handled safely as ordinary strings.

## Application behaviour expectations

- Invalid input should return a controlled response, not a database error.
- Known categories should be selected through allowlisted identifiers or safe query parameters.
- Unknown categories should return no results or a safe validation error.
- The application should not expose raw database error messages.

## Code-level checks

- Queries should use prepared statements / parameterized queries.
- No user-controlled value should be concatenated directly into SQL strings.
- ORM usage should avoid unsafe raw query construction.
- Database users should not have unnecessary permissions.

## Security finding regression checklist

For each SQLi fix, confirm:

- the vulnerable parameter no longer changes SQL behaviour,
- payloads are treated as values,
- sensitive data cannot be retrieved,
- administrator account takeover is no longer possible,
- tests exist to prevent the issue from returning.
