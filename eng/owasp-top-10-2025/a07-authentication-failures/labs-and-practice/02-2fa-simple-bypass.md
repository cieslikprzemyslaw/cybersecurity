# Lab: 2FA Simple Bypass

## Source

[PortSwigger Web Security Academy - 2FA simple bypass](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)

## Status

**PASS**

## Intended Authentication Flow

1. User submits username and password.
2. Backend verifies the first factor.
3. Backend creates a restricted `mfa_pending` state.
4. User submits a valid second factor.
5. Backend changes the state to `mfa_completed` or fully authenticated.
6. Protected pages and APIs become accessible.

## Identity Claim

The username claimed the Carlos identity.

## Authentication Evidence

- First factor: valid password.
- Second factor: MFA verification code.

Both factors were required by the intended flow.

## Observation

After valid username and password credentials, the application displayed the MFA step.

When navigation moved away from the MFA page, the application appeared to recognise the session as Carlos.

This was suspicious, but the UI alone was not sufficient proof.

## Controlled Test

Without submitting any MFA code, the session created after password verification was used to request the protected account page:

```http
GET /my-account
Cookie: session=<session-after-password-verification>
```

## Result and Evidence

The protected page returned Carlos's account without completion of the second factor.

This proved that the backend did not enforce MFA completion on the protected resource.

## Root Cause

The application created a sufficiently privileged authenticated session after password verification and failed to enforce the MFA-completed state on protected resources.

The weakness was not simply that a user could change the URL. Direct navigation only exposed the missing server-side state check.

## Required State Model

```text
unauthenticated
    -> password_verified / mfa_pending
    -> fully_authenticated / mfa_completed
```

The `mfa_pending` state must not have access to normal protected resources.

## STRIDE Category

**Spoofing** — an attacker with the victim's password could be accepted as fully authenticated without providing the required second factor.

## Impact

An attacker who has obtained valid username and password credentials can bypass the control intended to protect against credential compromise and access the victim's account.

## Security Requirement

> The server must not create or treat a session as fully authenticated until every required authentication factor has been verified, and every protected route and API must enforce the MFA-completed state.

## Remediation

The backend should:

- represent the pre-MFA state explicitly,
- restrict it to endpoints needed to complete or cancel authentication,
- bind MFA verification to the correct user, session, and attempt,
- enforce MFA state on all protected routes and APIs,
- expire and rate-limit MFA codes,
- rotate or upgrade the session safely after MFA completion,
- protect recovery, backup-code, trusted-device, and MFA-disablement flows.

## Regression Tests

- `/my-account` is rejected before MFA completion.
- Protected API endpoints are rejected before MFA completion.
- Direct navigation around `/login2` does not grant access.
- Invalid and expired codes do not complete authentication.
- Code for one account or attempt cannot complete another.
- Excessive MFA attempts trigger appropriate throttling.
- Successful MFA changes the server-side state to fully authenticated.

## Finding Summary

> An attacker with valid username and password credentials can bypass MFA because the application grants access to protected resources before the backend verifies completion of the second factor.
