# SQL Injection Regression Tests

## UNION-based SQLi regression tests

After fixing a UNION-based SQLi issue, test that:

- adding a single quote does not cause a database error or 500 response,
- adding `--` does not change query behaviour,
- `UNION SELECT NULL,NULL` is treated as a string or rejected safely,
- `UNION SELECT 'abc','def'` does not render injected values,
- attempting to select from `users` does not expose data,
- unknown categories return a safe response.

## Blind SQLi regression tests

For blind SQLi in cookies or other values, test that:

- TRUE and FALSE conditions do not affect visible markers,
- `Welcome back` or similar UI markers are not controlled by injected SQL logic,
- `TrackingId` or equivalent cookie values are handled as data,
- password length checks cannot be performed through response differences,
- character-by-character extraction is not possible.

## Code review regression checks

- No query string concatenation with untrusted input.
- Prepared statements are used consistently.
- ORM raw query usage is reviewed.
- Inputs with limited valid values use allowlists.
- Database user permissions are limited.
- SQL errors are not exposed to users.
