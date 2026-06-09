# Lab: Password Reset Broken Logic

## Source

[PortSwigger Web Security Academy - Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)

## Status

**PASS**

## Intended Authentication Flow

1. A user submits an account identifier to request password recovery.
2. The backend creates a reset token for that account.
3. The token is delivered through the user's recovery channel.
4. The user submits the token and a new password.
5. The backend verifies the token and resets only the account bound to it.

## Identity Claim

The `username` parameter selected the account whose password would be changed.

## Expected Authentication Evidence

`temp-forgot-password-token` was the recovery evidence. It should prove that the requester controls the recovery process for one specific account.

## Initial Observation

The reset submission contained separate client-controlled values:

```http
POST /forgot-password
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=<token-for-wiener>
&username=wiener
&new-password-1=<new-password>
&new-password-2=<new-password>
```

## Initial Mistake

I first wondered whether the browser `session` cookie sent with the reset request could authenticate me as Carlos.

A controlled test showed that the cookie was only the browser's current session. It was not the recovery token and did not automatically become Carlos's authenticated session.

## Hypothesis

The backend might validate that the reset token exists but trust the separate `username` parameter to select the target account.

## Controlled Test

The valid token remained unchanged while the target username was modified:

```http
POST /forgot-password
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=<valid-token-for-wiener>
&username=carlos
&new-password-1=<new-password>
&new-password-2=<new-password>
```

## Result

The server accepted the request and redirected.

This was evidence that the request was processed, but a redirect alone did not yet prove account takeover.

## Impact Confirmation

Logging in as Carlos with the newly selected password succeeded.

This confirmed:

- the token issued for Wiener was accepted,
- the client-controlled username retargeted the operation to Carlos,
- Carlos's password was changed,
- the attacker could authenticate as Carlos.

## Root Cause

The backend did not securely bind the password-reset token to the account whose password was changed. It relied on a client-controlled username to select the target identity.

The problem was not merely that a hidden form field could be edited. All browser-supplied data can be edited.

## STRIDE Category

**Spoofing** — the weakness allowed an attacker to replace another user's password and then authenticate as that user.

## Impact

Confirmed account takeover. If the target were privileged, the compromise could expose administrative data and functionality.

## Security Requirement

> The server must derive the target account from trusted reset-token state and must not rely on a client-controlled username to select the account whose password is changed.

## Remediation

Store and validate a server-side relation similar to:

```text
reset_token_hash
    -> user_id
    -> purpose=password_reset
    -> expires_at
    -> used_at
```

The server should:

- generate an unpredictable token,
- store only an appropriate protected representation such as a hash,
- derive the user from token state,
- reject expired, invalid, or used tokens,
- invalidate the token immediately after success,
- consider revoking existing sessions,
- avoid account enumeration in reset responses.

## Regression Tests

- Token for user A cannot reset user B.
- Modified or missing username cannot retarget the reset.
- Missing, empty, altered, expired, and used tokens are rejected.
- Token replay after successful reset is rejected.
- The new password works only for the account bound to the token.
- Existing sessions follow the documented revocation requirement.

## Finding Summary

> An attacker who knows a valid username can use their own password-reset token to reset another account because the backend does not verify that the token is bound to the selected user. This can lead to account takeover, including compromise of privileged accounts.
