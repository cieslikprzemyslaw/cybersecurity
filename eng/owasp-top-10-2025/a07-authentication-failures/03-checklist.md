# Authentication Review Checklist

Use this checklist during a frontend, API, middleware, CMS, or AppSec review.

## Identity and Flow

- [ ] Identify every authentication entry point: login, registration, verification, reset, recovery, MFA, remember-me, SSO fallback, support-assisted recovery.
- [ ] Document the identity claim used by each flow.
- [ ] Document the evidence expected from the user.
- [ ] Identify the backend component that verifies the evidence.
- [ ] Confirm that steps cannot be skipped, reordered, replayed, or called directly.
- [ ] Confirm that alternate endpoints enforce the same authentication rules.

## Password Login

- [ ] Existing and non-existing users receive equivalent client-facing errors.
- [ ] Status codes, response length, and timing do not create reliable enumeration.
- [ ] Failed attempts are throttled using layered controls rather than one weak signal.
- [ ] Lockout design does not allow easy denial of service.
- [ ] Known breached and weak passwords are rejected according to the product policy.
- [ ] Default and hard-coded credentials are not deployed.
- [ ] Authentication failures are logged without storing plaintext passwords.

## Password Reset and Recovery

- [ ] Reset responses do not reveal whether the account exists.
- [ ] Reset tokens are generated with sufficient unpredictability.
- [ ] Tokens expire after an appropriate period.
- [ ] Tokens are single-use.
- [ ] Tokens are bound to one account and one recovery purpose.
- [ ] The target account is derived from trusted token state.
- [ ] A client-controlled `username`, email, or user ID cannot retarget the reset.
- [ ] Reset links use a trusted HTTPS origin.
- [ ] Host/proxy headers cannot poison the reset URL.
- [ ] A successful reset invalidates the token immediately.
- [ ] Existing sessions are revoked or handled according to a documented risk decision.
- [ ] Resetting a password does not silently disable or bypass MFA.
- [ ] Recovery and support-assisted paths are at least as strong as normal login.

## MFA

- [ ] The application distinguishes `mfa_pending` from `mfa_completed`.
- [ ] Password verification alone does not create a fully authenticated session.
- [ ] Before MFA, the session can reach only endpoints required to finish authentication.
- [ ] Every protected route and API checks MFA completion server-side.
- [ ] Direct navigation cannot bypass MFA.
- [ ] MFA codes expire.
- [ ] MFA attempts are rate-limited.
- [ ] Codes are bound to the correct user and authentication attempt.
- [ ] Backup codes are protected, limited, and invalidated after use.
- [ ] Disabling MFA requires recent authentication.
- [ ] Trusted-device behaviour has explicit lifetime and revocation rules.

## Session Management

- [ ] A new unpredictable session identifier is issued after successful login.
- [ ] The identifier rotates after privilege changes or step-up authentication where appropriate.
- [ ] The anonymous/pre-auth session is not promoted without rotation.
- [ ] Logout invalidates the server-side session.
- [ ] Old sessions cannot be replayed after logout.
- [ ] Idle and absolute timeouts are configured.
- [ ] Remember-me tokens have appropriate lifetime, rotation, and revocation.
- [ ] Password reset, password change, account disablement, and suspected compromise have defined session-revocation behaviour.
- [ ] Concurrent sessions follow documented product requirements.
- [ ] Session identifiers do not appear in URLs or logs.

## Cookies and Browser State

- [ ] Session cookies use `HttpOnly` where JavaScript access is not required.
- [ ] Session cookies use `Secure`.
- [ ] `SameSite` is selected according to the application flow and CSRF design.
- [ ] `Path` and `Domain` are restricted to the required scope.
- [ ] Authentication is not decided from editable localStorage/sessionStorage values.
- [ ] Sensitive tokens are not unnecessarily persisted in browser storage.
- [ ] Frontend state is cleared on logout, while recognising that backend invalidation is the primary control.

## Sensitive Actions

- [ ] Password change requires current password or recent reauthentication where appropriate.
- [ ] Email, phone, MFA, payment, and recovery changes require step-up authentication when risk justifies it.
- [ ] Sensitive changes generate user notification.
- [ ] An authenticated session alone is not always treated as sufficient proof for high-risk actions.

## Logging and Detection

- [ ] Failed and successful authentication events are logged with useful context.
- [ ] Reset requests, token failures, and token replay are logged.
- [ ] MFA failures and unusual retry patterns can generate alerts.
- [ ] Credential stuffing and password spraying signals are monitored.
- [ ] Session creation, revocation, and suspicious reuse are observable.
- [ ] Logs avoid passwords, full reset tokens, and unnecessary secrets.

## Frontend Review

- [ ] Route guards are documented as UX controls, not security enforcement.
- [ ] Protected APIs reject unauthenticated and MFA-pending requests independently of the UI.
- [ ] Disabled controls do not replace server-side rate limiting or authorization.
- [ ] Client validation is repeated and enforced server-side.
- [ ] The UI does not expose account existence through avoidable error differences.
