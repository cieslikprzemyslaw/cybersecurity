# A07 Labs and Practice

## Completed Work

| Source | Exercise | Main lesson | Status |
|---|---|---|---|
| TryHackMe | OWASP Top 10 2025 - IAAA introduction and A07 section | Identification is a claim; authentication verifies evidence; authorization decides permissions; accountability records activity | PASS |
| PortSwigger | Password reset broken logic | Recovery evidence must be bound to the target account and the server must not trust a client-controlled username | PASS |
| PortSwigger | 2FA simple bypass | Password-verified state must remain restricted until the backend verifies MFA completion | PASS |
| Mentor checkpoint | Sessions, logout, enumeration, rate limiting | Session rotation and server-side invalidation are required; generic errors and layered throttling reduce credential attacks | PASS after correction |

## Detailed Notes

- [Password reset broken logic](labs-and-practice/01-password-reset-broken-logic.md)
- [2FA simple bypass](labs-and-practice/02-2fa-simple-bypass.md)
- [Practice summary](labs-and-practice/summary.md)

## Existing Related Practice

Username enumeration and authentication bypass were completed earlier and are linked rather than repeated:

- [Authentication Bypass and Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)

## Scope Honesty

The following topics were reviewed conceptually but were not completed as dedicated A07 exploitation labs in this session:

- session fixation,
- logout/session replay,
- concurrent session control,
- remember-me design,
- brute-force protection bypass,
- MFA code brute force,
- hard-coded and default credentials.

They are included in the checklist and regression tests because they are important review areas, but they are not presented as completed lab evidence.
