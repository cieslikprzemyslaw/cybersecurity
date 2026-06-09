# A07: Authentication Failures

This folder contains my practical notes for OWASP Top 10 2025 - A07: Authentication Failures.

## Status

**PASS — Frontend Developer transitioning into AppSec**

Completed work:

- TryHackMe: OWASP Top 10 2025 - IAAA introduction and A07 Authentication Failures section,
- PortSwigger: Password reset broken logic,
- PortSwigger: 2FA simple bypass,
- short review of authentication, recovery, session lifecycle, account enumeration, rate limiting, and logout behaviour.

The status does not mean expert-level IAM knowledge. It means I can review common authentication flows, identify missing server-side verification, separate facts from assumptions, and communicate a useful finding.

## Start Here

- [Overview](01-overview.md)
- [Labs and practice](02-labs-or-practice.md)
- [Authentication review checklist](03-checklist.md)
- [Regression test ideas](04-regression-tests.md)
- [Learning notes](05-learning-notes.md)
- [Detailed lab notes](labs-and-practice/README.md)
- [Example security findings](security-findings/)

## Core Mental Model

```text
identity claim
    -> authentication evidence
    -> server-side verification
    -> authenticated or MFA-pending session
    -> session lifecycle and invalidation
```

The main review questions are:

1. What identity is being claimed?
2. What evidence is meant to prove that identity?
3. Which server-side component verifies the evidence?
4. Is the evidence bound to the correct user, action, and authentication attempt?
5. Can a step be skipped, repeated, reordered, or modified?
6. When is the session created or rotated?
7. How is the session invalidated?
8. Can recovery or fallback mechanisms bypass the primary login controls?

## Frontend Perspective

React route guards, hidden components, redirects, disabled buttons, and client-side validation support usability, but they are not authentication boundaries.

The browser and all values sent from it are under user control. The backend must verify credentials, reset tokens, MFA completion, sessions, and access to protected endpoints.

## Completed Practical Patterns

### Password reset token not bound to the target account

A valid reset token issued for one account was accepted while a client-controlled `username` selected another account. The password of the second account was changed and the new password successfully authenticated to it.

### MFA step displayed but not enforced

After valid username and password verification, the application displayed an MFA page. However, the session could access `/my-account` before an MFA code was submitted. The backend did not enforce an MFA-completed state on the protected resource.

## Related Existing Notes

Username enumeration and earlier authentication bypass practice are linked instead of duplicated:

- [Authentication Bypass and Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)

## Direct Learning Links

- [OWASP A07:2025 - Authentication Failures](https://owasp.org/Top10/2025/A07_2025-Authentication_Failures/)
- [TryHackMe - OWASP Top 10 2025: IAAA Failures](https://tryhackme.com/room/owasptopten2025one)
- [PortSwigger - Authentication vulnerabilities](https://portswigger.net/web-security/authentication)
- [PortSwigger Lab - Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)
- [PortSwigger Lab - 2FA simple bypass](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)

## File Roles

- `01-overview.md` is the concise source of truth for the A07 concepts covered.
- `02-labs-or-practice.md` indexes the completed work without duplicating all evidence.
- `03-checklist.md` is a developer/AppSec authentication review checklist.
- `04-regression-tests.md` contains reusable tests for fixes.
- `05-learning-notes.md` records my reasoning, initial mistakes, and corrections.
- `labs-and-practice/` contains detailed practical evidence.
- `security-findings/` contains developer-facing findings.
