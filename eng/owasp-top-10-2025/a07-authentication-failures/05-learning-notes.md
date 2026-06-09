# A07 Learning Notes

## Starting Point

I already understood basic login flows, cookies, tokens, username enumeration, and the difference between authentication and authorization at a general level.

My goal was not to become an identity architect. The goal was to build a practical foundation for reviewing common application flows as a Frontend Developer transitioning into AppSec.

## What I Initially Explained Correctly

- Identification selects or claims an identity.
- Authentication provides evidence for that identity.
- Authorization decides what the authenticated user can do.
- Accountability records who did what and when.
- Client-side controls are not enough to secure authentication.

## Corrections That Improved My Understanding

### A username is a claim, not proof

I first described identity mainly as an ID or username. The important correction was that this value only says:

> I claim to be this account.

The password, MFA factor, recovery token, or valid session is the evidence the backend must verify.

### A cookie is not automatically another user's session

During the password-reset lab, I initially wondered whether the browser `session` cookie became a session for Carlos because the reset request contained `username=carlos`.

A controlled request to `/my-account` showed that this assumption was not supported. The cookie represented the browser's current session, not authenticated proof for Carlos.

This was useful because it separated:

- a hypothesis about the cookie,
- from the actual recovery evidence in the reset link.

### Session handling after login means rotation, not only temporary validity

I initially answered that a successful login should extend or temporarily validate the session.

The important correction was:

- the backend should issue or rotate to a new unpredictable session identifier,
- bind it to the authenticated user,
- apply an appropriate lifetime.

This prevents an anonymous or attacker-influenced identifier from continuing after authentication.

### Logout testing means replaying the old session

I first described the test as trying to log in with the same token. More precisely:

1. save the authenticated session,
2. log out,
3. replay the saved session against a protected endpoint,
4. confirm the backend rejects it.

### MFA is a state transition, not a page

The 2FA lab made the strongest practical point:

- the application displayed an MFA page,
- but the backend already allowed the password-verified session to reach `/my-account`.

The weakness was not simply that the URL could be changed. The server failed to enforce that MFA had been completed.

A clearer state model is:

```text
unauthenticated
    -> password_verified / mfa_pending
    -> fully_authenticated / mfa_completed
```

### Password reset is alternative authentication

I initially struggled to explain why MFA does not automatically fix an insecure reset flow.

The correction was that password reset is a separate route for proving control of an account. If recovery evidence is weak or incorrectly bound, normal login MFA does not repair that recovery weakness.

## Password Reset Lab Reasoning

### Intended flow

1. User requests a reset.
2. Server issues a token for that account.
3. User receives the token through the recovery channel.
4. Server verifies the token.
5. Server resets only the account bound to the token.

### What I controlled

```text
temp-forgot-password-token=<valid-token-for-wiener>
username=carlos
new-password-1=<new-password>
new-password-2=<new-password>
```

### Evidence progression

- `302` showed that the server accepted the reset request.
- It did not yet prove account takeover.
- Successful login as Carlos using the new password confirmed the impact.

### Final understanding

The root cause was not that a hidden form field could be edited. All client data can be edited.

The root cause was that the backend did not verify that the recovery evidence authorised resetting the selected account.

## 2FA Lab Reasoning

### Intended flow

1. Verify username and password.
2. Create a restricted MFA-pending state.
3. Verify the second factor.
4. Only then grant access to protected resources.

### Observed behaviour

After valid credentials but before any MFA code, the session could access `/my-account` as Carlos.

### Final understanding

The frontend displayed step two, but the protected resource did not enforce the MFA-completed state.

## Facts Versus Assumptions

A useful lesson from both labs was to avoid declaring impact too early.

### Password reset

- Fact: server returned `302` after the modified reset request.
- Assumption: Carlos's password may have changed.
- Evidence: login as Carlos using the new password succeeded.
- Impact: confirmed account takeover.

### MFA

- Fact: a session existed after password verification.
- Observation: the MFA page was displayed.
- Evidence: `/my-account` returned Carlos's account without an MFA code.
- Impact: confirmed MFA bypass for an attacker with valid credentials.

## Findings I Can Now Explain

### Password reset

> An attacker who knows a valid username can use their own password-reset token to reset another account because the backend does not verify that the token is bound to the selected user. This can lead to account takeover, including compromise of privileged accounts.

### MFA

> An attacker with valid username and password credentials can bypass MFA because the application grants access to protected resources before the backend verifies completion of the second factor.

## Current Limits

I understand the review approach and core controls, but I have not completed dedicated practical A07 labs for every session-management or credential-attack pattern.

Future practice can deepen:

- session fixation,
- session invalidation across devices,
- remember-me token security,
- MFA code brute force,
- complex rate-limit bypasses,
- SSO/OIDC-specific validation.

These are next-depth topics, not blockers for the current FE-to-AppSec foundation.

## Interview-Style Takeaways

- Identification claims an identity; authentication verifies evidence; authorization checks permissions.
- A recovery token is authentication evidence and must be bound to one account.
- MFA must be enforced by backend state on every protected resource.
- A React route guard is useful for UX but is not a security boundary.
- Session identifiers should rotate after authentication and be invalidated server-side on logout.
- A suspicious response is not always proof; impact should be confirmed with a controlled test.

## Final Status

**PASS — practical foundation appropriate for a Frontend Developer transitioning into AppSec.**
