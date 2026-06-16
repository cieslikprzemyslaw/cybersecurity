# Security Events and Audit Logs

## Choosing events by security question

More logging is not automatically better. Each event should help answer a defined security question such as:

- Is an account being attacked?
- Was a protected resource denied correctly?
- Who changed a user's privileges?
- Was an invalid or revoked token reused?
- Did a file fail malware validation?
- Was a sensitive export created?
- Did a user bypass an expected business sequence?

## Events commonly worth logging

### Authentication and session events

- successful and failed login,
- successful and failed MFA,
- MFA recovery or reset,
- password-reset request, token validation, completion, and failure,
- session creation, renewal, expiry, logout, and invalidation,
- reuse of an expired, revoked, or invalid token,
- successful login after repeated failures.

### Authorization and privilege events

- authorization denial,
- role or permission change,
- administrative action,
- privileged-user creation or deletion,
- use of shared, default, emergency, or break-glass access,
- direct access attempts to protected objects.

### Data and business events

- access to highly sensitive records,
- sensitive create, read, update, delete, import, or export,
- high-value transaction,
- unexpected state transition,
- workflow step performed out of order,
- attempt to exceed a business limit,
- action that does not make sense in the business context.

### Validation and system events

- server-side input-validation failure,
- output-validation failure,
- unexpected HTTP method,
- suspicious JWT validation failure,
- rate-limit trigger,
- file-upload validation and malware-scan result,
- deserialization failure,
- backend TLS or certificate-validation failure,
- security-control error,
- logging or monitoring disabled,
- application startup, shutdown, crash, or logging initialisation.

## Log both relevant success and failure

Only logging failures creates an incomplete picture. Important successful events may include:

- login success after repeated failures,
- successful password reset,
- successful role change,
- successful sensitive-data export,
- successful use of emergency access,
- successful administrative deletion.

A successful action may be the strongest indicator that an earlier attack attempt finally worked.

## Useful event structure

Not every event needs every field. Select the minimum fields needed for monitoring and investigation.

Common fields include:

```text
timestamp
eventName
severity
application / service / version
environment
actorUserId
targetUserId or affectedResource
action
result
safeReasonCode
requestId / correlationId / interactionId
sourceIp
userAgent
httpMethod / route / statusCode
securityControlResult
confidence
responseTaken
```

The event should normally answer:

```text
when + where + who + what + result
```

## Event type, result, and severity are separate

Example:

```json
{
  "eventName": "authn_login_fail",
  "result": "failure",
  "severity": "WARN"
}
```

- `eventName` describes what happened.
- `result` describes the outcome.
- `severity` describes the event's importance and response priority in context.

`FAILED` is an outcome, not a universal severity level.

## Vocabulary examples

A consistent vocabulary makes cross-service detection easier. Useful examples from the OWASP Logging Vocabulary Cheat Sheet include:

```text
authn_login_success
authn_login_successafterfail
authn_login_fail
authn_login_fail_max
authn_login_lock
authn_password_change
authn_password_change_fail
authn_token_created
authn_token_revoked
authn_token_reuse
authz_fail
authz_change
authz_admin
excess_rate_limit_exceeded
upload_validation
input_validation_fail
malicious_sqli
malicious_direct_reference
privilege_permissions_changed
sensitive_create
sensitive_read
sensitive_update
sensitive_delete
session_created
session_renewed
session_expired
session_use_after_expire
sys_startup
sys_shutdown
sys_crash
sys_monitor_disabled
user_created
user_updated
user_archived
user_deleted
```

The vocabulary is a starting point, not a complete detection policy. An organisation still needs:

- documented field names and types,
- consistent timestamps,
- severity rules,
- thresholds and time windows,
- ownership and response playbooks.

## Severity examples

The OWASP vocabulary uses example levels such as:

- `INFO` for expected but security-relevant activity,
- `WARN` for suspicious, policy-relevant, or administratively sensitive activity,
- `CRITICAL` for high-confidence, high-risk indicators.

These are not universal mappings. Severity should consider:

- asset sensitivity,
- event result,
- confidence,
- frequency,
- account privilege,
- business impact,
- expected response urgency.

## Audit event example: role change

```ts
logger.warn("User role changed", {
  timestamp: new Date().toISOString(),
  eventName: "privilege_permissions_changed",
  actorUserId: authenticatedUser.id,
  targetUserId: request.body.userId,
  previousRole,
  newRole: request.body.role,
  result: "success",
  requestId: request.id,
});
```

The internal actor and target identifiers are evidence, not secrets. Passwords, access tokens, session cookies, and the complete request body are not required to prove the role change.

## Audit-log integrity

High-value audit trails should be protected against unauthorised:

- reading,
- modification,
- deletion,
- truncation,
- reordering,
- clock manipulation.

Possible controls include:

- append-oriented or immutable storage,
- strict write and read permissions,
- central collection,
- protected transport,
- retention and backup policies,
- monitoring for logging stoppage or deletion attempts,
- separation between application writers and log readers or administrators.
