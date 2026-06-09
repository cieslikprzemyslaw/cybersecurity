# A07:2025 - Authentication Failures

## What This Category Means

Authentication answers:

> Who is this user, and has the system received sufficient proof of that identity?

An Authentication Failure occurs when an application accepts insufficient or incorrect proof, allows a required step to be skipped, trusts user-controlled identity data, mishandles authentication state, or provides an insecure recovery path.

OWASP A07 includes weaknesses such as ineffective protection against automated credential attacks, weak recovery, missing or ineffective MFA, session fixation, session identifiers that are not rotated, and sessions that are not invalidated correctly.

## Critical Distinctions

### Identification

Identification is an identity claim, for example:

```text
username=carlos
email=user@example.com
userId=123
```

It answers: **Which identity is being claimed?**

A username or email address is not proof of identity.

### Authentication

Authentication verifies evidence that should prove the claim, for example:

- password,
- MFA code,
- password-reset token,
- passkey or hardware token,
- an existing authenticated session.

A session cookie or JWT usually represents that authentication happened earlier. It must still be validated by the backend.

### Authorization

Authorization decides what an authenticated identity may access or do.

A user may be authenticated correctly but still exploit Broken Access Control if the application permits an action outside their permissions.

### Accountability

Accountability provides an audit trail showing who performed an action, what happened, and when or from where it happened. Logging supports detection and investigation but does not replace prevention.

### Session management

Session management maintains authenticated state after login. It includes session creation, rotation, expiration, invalidation, concurrent sessions, remember-me behaviour, and logout.

### Password reset versus password change

- **Password reset** is an alternative authentication/recovery mechanism used when the current password is unavailable.
- **Password change** normally happens inside an authenticated session and should often require the current password or recent reauthentication.

### MFA versus step-up authentication

- **MFA** requires independent factors as part of authentication.
- **Step-up authentication** requires additional proof before a sensitive action, even when a session already exists.

## Authentication Review Flow

For each flow, review:

1. **Identity claim** — which account is selected?
2. **Evidence** — what is meant to prove the identity?
3. **Verification** — what must the server check?
4. **Binding** — is the evidence bound to the correct account, action, and attempt?
5. **Lifetime** — when does it expire?
6. **Single use** — can it be replayed?
7. **Session creation** — what state exists after each step?
8. **Session rotation** — does the identifier change after login or privilege change?
9. **Invalidation** — what happens on logout, reset, password change, or account disablement?
10. **Failure handling** — are responses generic, rate-limited, logged, and fail-closed?
11. **Abuse cases** — can steps be skipped, reordered, repeated, or modified?
12. **Regression tests** — how will the fix remain testable?

## Password Authentication and Credential Attacks

### Username enumeration

Different behaviour for existing and non-existing accounts can reveal valid identities. Signals may include:

- different error messages,
- different status codes,
- different response lengths,
- timing differences,
- account-lock behaviour.

Enumeration is useful to attackers because it creates a reliable target list for password spraying, credential stuffing, phishing, or recovery abuse.

### Brute force

Brute force repeatedly guesses credentials for one or more accounts.

### Password spraying

Password spraying tries a small set of common passwords across many accounts to reduce account-lock risk.

### Credential stuffing

Credential stuffing tests username/password pairs obtained from previous breaches.

### Rate limiting and account locking

A single control is rarely sufficient:

- IP-only controls can be bypassed through distributed sources and can affect shared networks.
- Account-only locking can be abused to cause denial of service.
- Permanent lockout can let an attacker block legitimate users.

A layered design may combine per-account and per-source throttling, increasing delays, risk signals, MFA, monitoring, and user notification.

## Password Reset and Recovery

Password reset is another route into an account. It must be at least as secure as the primary login flow.

A secure reset token should be:

- unpredictable,
- time-limited,
- single-use,
- bound to one account and one recovery purpose,
- invalidated after successful use,
- transmitted through a trusted origin and protected channel.

The server should derive the target account from trusted token state. It should not rely on a separate client-controlled username to decide which password is changed.

After a successful reset, the application should consider invalidating existing sessions, especially when the reset may indicate account compromise.

## Multi-Factor Authentication

MFA is a complete state transition, not only a code input page.

A safe flow distinguishes:

```text
unauthenticated
    -> password_verified / mfa_pending
    -> fully_authenticated / mfa_completed
```

Before MFA completion, the session should only reach endpoints required to finish authentication. Every protected route and API must verify the MFA-completed state server-side.

MFA codes should be:

- short-lived,
- rate-limited,
- bound to the correct user and authentication attempt,
- invalid after successful use when appropriate.

Backup codes, account recovery, trusted devices, and MFA disablement must not create a weaker path around MFA.

## Session Management

### Session fixation

Session fixation occurs when an attacker controls or predicts a session identifier before authentication and the application keeps using the same identifier after the victim logs in.

### Session rotation

After successful authentication or a privilege change, the server should issue a new unpredictable session identifier. Merely extending the lifetime of the original anonymous session is not enough.

### Logout

Logout should invalidate the authenticated session server-side. Removing browser state alone is not sufficient.

A practical test is to save the session cookie, log out, and then replay the old cookie against a protected endpoint.

### Expiration

Sessions should have appropriate:

- idle timeout,
- absolute timeout,
- remember-me lifetime,
- revocation behaviour after password reset, password change, or account disablement.

### Cookie attributes

- `HttpOnly` reduces direct JavaScript access to a cookie.
- `Secure` restricts transmission to HTTPS.
- `SameSite` influences cross-site cookie sending and helps reduce some CSRF risk.
- `Path` and `Domain` should be no broader than required.

These flags are defence in depth. They do not repair missing token validation, MFA bypass, session fixation, or failed server-side invalidation.

## Frontend Responsibilities and Limits

Useful frontend controls include:

- good usability and clear feedback,
- early validation,
- reducing accidental mistakes,
- clearing sensitive in-memory state,
- avoiding unnecessary storage of tokens,
- safely handling redirects and error messages.

However:

- React route guards are not a security boundary.
- A hidden component does not protect its API.
- A disabled button does not enforce rate limiting.
- `localStorage` and `sessionStorage` values are user-controlled.
- Redirecting away from a protected page does not protect the endpoint.
- The backend must verify authentication state for every protected request.

## Facts, Assumptions, Evidence, and Impact

### Facts

Directly observed behaviour such as requests, responses, cookies, state changes, and successful authentication.

### Assumptions

A possible backend explanation that has not yet been proven.

### Evidence

A reproducible result showing that the expected authentication control can be bypassed or is missing.

### Impact

The access or action made possible by the weakness.

A `302` after a reset request showed that the request was accepted, but successful login with the new password was the evidence that confirmed account takeover.

## STRIDE Mapping

Both completed practical findings map primarily to **Spoofing** because the weaknesses allowed an attacker to be accepted as another identity:

- reset-token/account binding failure allowed authentication as the victim after password replacement,
- MFA enforcement failure allowed a password-verified attacker to be treated as fully authenticated.

## Security Requirements

Examples of specific requirements:

- The server must not create a fully authenticated session until every required factor is verified.
- Password reset tokens must be unpredictable, time-limited, single-use, and bound to one account.
- The account being reset must be derived from trusted server-side token state.
- The session identifier must rotate after successful authentication.
- Logout must invalidate the server-side session.
- Sensitive account changes must require recent authentication where appropriate.
- Authentication responses must not reveal whether an account exists.
- Authentication controls must be enforced consistently across all routes and APIs.
