# MFA Completion Is Not Enforced on Protected Resources

## Summary

After the application verifies the username and password, it creates a session that can access protected resources before the second authentication factor is completed.

## Severity

**High** in the tested scenario because an attacker with compromised credentials can bypass the control intended to prevent account access using only the password.

## STRIDE

**Spoofing** — the application treats the attacker as fully authenticated without all required identity evidence.

## Affected Flow

Two-factor authentication and access to the protected account.

## Preconditions

The attacker knows a valid username and password but does not possess the MFA code.

## Evidence

Without submitting an MFA code, the password-verified session successfully accessed `/my-account` on behalf of the victim.

## Reproduction

1. Submit the victim's valid username and password.
2. Stop at the MFA page without submitting a code.
3. Use the current session to request a protected resource.
4. Observe that the account page is returned.

## Root Cause

The backend does not distinguish or enforce `mfa_pending` and `mfa_completed` states for protected resources.

## Impact

Stolen or reused credentials are sufficient to access the account even though MFA appears to be enabled.

## Security Requirement

> Every protected route and API must require a fully authenticated server-side session with all mandatory factors completed.

## Remediation

- Create an explicit restricted pre-auth / MFA-pending state.
- Allow that state to access only authentication-completion endpoints.
- Enforce the MFA-completed state on every protected route and API.
- Bind MFA codes to the user, session, and authentication attempt.
- Apply code expiry and rate limiting.
- Safely rotate or upgrade the session after successful MFA.

## Regression Tests

- Protected pages reject MFA-pending sessions.
- Protected APIs reject MFA-pending sessions.
- Direct navigation cannot bypass MFA.
- Invalid or expired codes cannot create a fully authenticated session.
- Successful MFA is the only transition to the fully authenticated state.
