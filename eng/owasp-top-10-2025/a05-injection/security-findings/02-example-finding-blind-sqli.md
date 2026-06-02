# Example Finding: Boolean-Based Blind SQL Injection in TrackingId Cookie

## Summary

The `TrackingId` cookie is vulnerable to boolean-based blind SQL Injection. An attacker can infer database information by observing whether the `Welcome back` message appears in the response.

## Affected parameter

```text
Cookie: TrackingId=...
```

## Evidence

The following behaviour was observed:

```text
Normal TrackingId -> Welcome back appears
TRUE SQL condition -> Welcome back appears
FALSE SQL condition -> Welcome back disappears
```

This shows that the cookie value influences a backend SQL query and that response differences can be used as a true/false oracle.

Additional checks confirmed:

```text
users table exists
administrator user exists
administrator password length = 20
password can be extracted character by character
```

## Impact

An attacker can extract sensitive data without the application displaying database results directly. In the lab scenario, the administrator password was recovered and used to access the administrator account.

## Root cause

The application likely uses the `TrackingId` cookie in a SQL query without parameterization.

## Recommendation

- Use prepared statements / parameterized queries.
- Treat cookie values as untrusted input.
- Avoid unsafe raw SQL in tracking/session logic.
- Use safe ORM patterns.
- Apply least privilege to database users.
- Ensure response behaviour does not reveal query truth values.
- Add regression tests for boolean-based SQLi patterns.

## Regression tests

- TRUE and FALSE conditions in `TrackingId` should not change the response marker.
- The `Welcome back` message should not depend on injectable SQL logic.
- Password length inference should not be possible.
- Character-by-character extraction should not be possible through repeated requests.
