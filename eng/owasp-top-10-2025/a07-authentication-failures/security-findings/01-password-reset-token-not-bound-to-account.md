# Password Reset Token Not Bound to Target Account

## Summary

The password-reset endpoint accepts a valid reset token while allowing a separate client-controlled `username` parameter to select another account. The server does not verify that the token was issued for the account whose password is changed.

## Severity

**High** in the tested scenario because the weakness resulted in confirmed account takeover.

Final severity in a real application should consider account privileges, MFA/recovery behaviour, session revocation, exposure of valid usernames, and compensating controls.

## STRIDE

**Spoofing** — the attacker can replace a victim's password and authenticate as the victim.

## Affected Flow

Password reset / account recovery.

## Preconditions

- Attacker can obtain a valid reset token for their own account.
- Attacker knows or can identify the victim's username.

## Evidence

A token issued for one user was submitted with another user's username. The server accepted the reset, and the new password successfully authenticated to the victim account.

## Reproduction

1. Request a password reset for an attacker-controlled account.
2. Obtain the valid reset token.
3. Submit the password-reset request while changing the target username to the victim.
4. Set a new password.
5. Authenticate to the victim account with the new password.

## Root Cause

The reset token is not securely bound to the target account. The server trusts a client-controlled identity field during a security-sensitive state change.

## Impact

An attacker can take over another account. A privileged target could lead to administrative compromise and access to sensitive application data or functionality.

## Security Requirement

> The server must derive the password-reset target from trusted token state and reject any attempt to use the token for a different account.

## Remediation

- Store a server-side mapping from token to user, purpose, expiry, and use status.
- Validate the token before changing any password.
- Derive the user from the validated token rather than a request username.
- Make tokens unpredictable, short-lived, and single-use.
- Invalidate the token immediately after success.
- Review session revocation after password reset.
- Return generic recovery responses to reduce enumeration.

## Regression Tests

- Token for A cannot reset B.
- Altered username does not retarget the operation.
- Empty, missing, invalid, expired, and used tokens are rejected.
- Token replay is rejected.
- Successful reset affects only the token-bound account.
