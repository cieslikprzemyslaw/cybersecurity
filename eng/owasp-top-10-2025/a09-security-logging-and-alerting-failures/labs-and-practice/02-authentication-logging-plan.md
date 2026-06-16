# Authentication Logging and Alerting Plan

## Scenario

```text
three failed logins
    -> successful password verification
    -> two failed MFA attempts
    -> successful MFA
    -> authenticated session created
```

## Authoritative source

The Identity or Auth Service should emit the authoritative authentication events because it verifies credentials, MFA completion, and session state.

Frontend telemetry may record UX errors or a request ID, but it cannot prove that authentication succeeded.

## Expected events

```text
authn_login_fail
authn_login_fail
authn_login_fail
authn_login_success
authn_mfa_fail
authn_mfa_fail
authn_mfa_success
session_created
```

## Example failed-login event

```json
{
  "timestamp": "2026-06-16T12:00:00Z",
  "eventName": "authn_login_fail",
  "accountId": "usr_123",
  "sourceIp": "203.0.113.24",
  "userAgent": "Mozilla/5.0",
  "requestId": "req_a87f",
  "result": "failure",
  "reasonCode": "INVALID_CREDENTIALS"
}
```

The example has eight fields. HTTP status or route could be included when they answer a defined investigation need, but query strings must not contain credentials.

## Data exclusions

Do not log:

- password,
- MFA code,
- session cookie,
- access or refresh token,
- reset token,
- complete `Authorization` header,
- full login request body.

## Detection rule 1: repeated failures

```text
Pattern: authn_login_fail
Correlation: same account
Threshold: 5 failures
Window: 5 minutes
Severity: WARN
Owner: security team or on-call owner
Response: review sources, rate limits, account risk, and related activity
```

## Detection rule 2: success after failures

```text
Pattern: authn_login_success after at least 5 authn_login_fail events
Correlation: same account
Window: 10 minutes
Severity: HIGH; higher for a privileged account if policy requires
Owner: security team or on-call owner
Response: review MFA, session, source context, and post-login actions; contain when confidence is high
```

## Learning correction

My first answer listed login and MFA events but did not cover the session lifecycle or define a full detection rule. I also needed to separate a single event from a correlated pattern.

## Main takeaway

> The Auth Service emits evidence. A detection rule turns a suspicious sequence of that evidence into an actionable alert.
