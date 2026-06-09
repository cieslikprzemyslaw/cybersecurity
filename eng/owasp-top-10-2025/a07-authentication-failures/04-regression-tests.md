# Authentication Regression Tests

Select tests that match the feature and finding. Not every application requires every test.

## Password Reset and Recovery

### Token validation

- Missing reset token is rejected.
- Empty reset token is rejected.
- Modified reset token is rejected.
- Expired token is rejected.
- Already-used token is rejected.
- Token replay after a successful reset is rejected.

### Account binding

- Token for user A cannot reset user B.
- Changing the client-controlled username does not retarget the reset.
- Removing the username does not cause an unsafe fallback.
- The server derives the target account from trusted token state.
- Token issued for one purpose cannot be used for another account action.

### Lifecycle

- Successful reset invalidates the token immediately.
- Existing sessions are revoked according to the documented requirement.
- Old password no longer authenticates after reset.
- New password authenticates only to the intended account.
- Reset response remains generic for existing and non-existing accounts.

## MFA

### State enforcement

- After password verification, session state is `mfa_pending`, not fully authenticated.
- `/my-account` is rejected or redirected before MFA completion.
- Protected API calls are rejected before MFA completion.
- Direct navigation around the MFA page does not grant access.
- Alternate protected routes enforce the same MFA state.

### Code handling

- Invalid MFA code is rejected.
- Expired MFA code is rejected.
- Code for user A cannot complete user B's authentication attempt.
- Excessive attempts trigger appropriate throttling.
- Successful verification changes state to `mfa_completed`.
- Reuse of a code follows the intended one-time-use policy.

### Recovery and management

- Backup/recovery flow does not bypass required proof.
- Disabling MFA requires recent authentication.
- Trusted-device token expires and can be revoked.

## Session Management

### Rotation

- Record the session identifier before login.
- Log in successfully.
- Confirm that the identifier changes after authentication.
- Confirm appropriate rotation after privilege elevation or step-up authentication.

### Logout and invalidation

- Save an authenticated session cookie.
- Log out normally.
- Replay the old cookie against a protected endpoint.
- Confirm that the backend rejects it.
- Repeat for all relevant session/token types.

### Password events

- Change password and test whether old sessions behave according to requirements.
- Reset password and test whether old sessions behave according to requirements.
- Disable the account and confirm active sessions are revoked.

### Timeout

- Confirm idle timeout.
- Confirm absolute timeout.
- Confirm remember-me duration.
- Confirm expired tokens fail closed.

## Enumeration and Credential Attack Controls

- Compare existing-user/wrong-password and non-existing-user responses.
- Compare message, status code, response length, and timing.
- Test rate limiting across the primary login endpoint.
- Test alternate login/API/mobile endpoints for consistent controls.
- Test whether source-IP changes trivially bypass controls.
- Test whether account locking can be abused to deny service.
- Verify that suspicious patterns are logged and alertable.

## Frontend/API Consistency

- Bypass React navigation and call the protected API directly.
- Modify localStorage/sessionStorage authentication indicators.
- Confirm that editable browser state cannot create authentication.
- Remove or alter client-supplied identity fields.
- Confirm that backend decisions remain correct when the frontend flow is skipped.
