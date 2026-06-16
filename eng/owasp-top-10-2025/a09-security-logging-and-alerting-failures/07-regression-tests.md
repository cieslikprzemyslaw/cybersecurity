# A09 Regression Tests

Select tests relevant to the feature being reviewed.

## Authentication and session events

### Successful authentication

Perform a valid login.

Expected:

- one `authn_login_success` event,
- correct account ID, timestamp, result, service, and request ID,
- no password, token, cookie, or full request body.

### Failed authentication

Submit invalid credentials.

Expected:

- one `authn_login_fail` event,
- safe reason code,
- generic user-facing error,
- no account enumeration through the response or log exposure.

### Repeated failure detection

Generate the configured number of failures inside the rule window.

Expected:

- individual events remain available,
- correlation rule matches at the defined threshold,
- one useful alert is generated rather than one alert per event,
- alert contains account, source context, time range, and rule name.

### Success after failures

Generate repeated failures followed by a successful login.

Expected:

- success event is correlated with prior failures,
- severity is increased according to policy,
- alert reaches the named owner,
- playbook identifies session and post-login activity for review.

### MFA and recovery

Test:

- MFA failure,
- repeated MFA failure,
- MFA recovery,
- password-reset request and completion,
- invalid or reused reset token.

Expected:

- separate event types and results,
- no MFA code, recovery code, or reset token in logs,
- high-risk sequences generate the expected detection.

### Invalidated session

Reuse an expired or revoked session.

Expected:

- request is rejected,
- `session_use_after_expire` or equivalent event is emitted,
- repeated reuse can be correlated and alerted.

## Authorization and audit events

### Role change audit record

Change a user's role.

Expected event contains:

```text
timestamp
eventName
actorUserId
targetUserId
previousRole
newRole
result
requestId
```

Expected exclusions:

```text
password
sessionCookie
accessToken
complete requestBody
```

### Failed role change

Attempt the same change without permission.

Expected:

- authorization denial event,
- no role change,
- target and attempted action identified safely,
- no sensitive record contents.

### High-risk action without an audit event

Temporarily remove or break event emission in a test environment.

Expected:

- automated test fails,
- deployment or quality gate identifies the missing audit requirement.

## Log injection

### Newline and control characters

Submit a username, filename, or header containing newline and control characters.

Expected:

- exactly one valid structured event,
- input remains inside one field,
- event name, severity, result, and reason code remain application-controlled,
- no forged second entry,
- downstream parser and viewer remain safe.

### Oversized field

Submit an extremely long loggable value.

Expected:

- documented truncation or rejection,
- no storage or parser failure,
- original event type and metadata preserved.

## Sensitive-data leakage

### Secret scanning of captured logs

Exercise login, password reset, MFA, upload, and role-change flows, then inspect captured events.

Expected absence of:

- passwords,
- session IDs and cookies,
- access and refresh tokens,
- reset tokens,
- MFA secrets and codes,
- API keys,
- private keys,
- complete authorization headers.

### Frontend telemetry redaction

Trigger a client error while the page contains sensitive form data or API responses.

Expected:

- approved metadata and correlation ID only,
- no sensitive DOM snapshot, form value, token, cookie, or response body.

## Collection, availability, and integrity

### Collector or network failure

Simulate loss of the remote collector or network connectivity.

Expected:

- security control does not fail open,
- event loss or buffering follows documented policy,
- operational signal indicates degraded logging,
- application does not expose internal errors to the user.

### Storage exhaustion

Simulate disk or ingestion capacity exhaustion.

Expected:

- application remains within defined availability behaviour,
- logging failure is detected,
- security events are not silently discarded without an operational signal.

### Unauthorised log access

Attempt to read logs as an unauthorised user or service.

Expected:

- access denied,
- attempt is logged where appropriate,
- sensitive events are not exposed through a web-accessible path.

### Modification or deletion attempt

Attempt to alter or delete protected audit records.

Expected:

- operation denied or detected,
- evidence of the attempt is preserved separately,
- alert generated according to policy.

### Timestamp consistency

Generate events across relevant services.

Expected:

- ISO 8601 timestamps with UTC offsets,
- ordering and clock differences remain understandable,
- event time and ingestion time are distinguished where needed.

## Alert delivery and quality

### Alert channel failure

Simulate a failed notification channel.

Expected:

- delivery failure is detected,
- fallback or escalation path follows policy,
- the detection event remains stored.

### Duplicate alerts

Generate many matching events for the same incident.

Expected:

- deduplication or grouping works as designed,
- evidence is retained without overwhelming the owner.

### DAST and security testing

Run an approved scan or controlled test that should match a detection rule.

Expected:

- events are classified as authorised testing where appropriate,
- events are not silently excluded,
- expected rule and alert still work,
- test evidence documents the full event-to-response chain.
