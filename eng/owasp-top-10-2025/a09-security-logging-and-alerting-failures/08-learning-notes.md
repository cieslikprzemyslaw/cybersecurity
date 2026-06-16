# Learning Notes

## Initial understanding

I understood that logs help detect suspicious activity, but my first mental model was incomplete.

I initially thought logging was mostly a separate monitoring product and that a backend would simply create a log and send it to a database. The improved understanding is:

```text
application or security-control component
    -> structured event
    -> collector and protected storage
    -> detection and correlation
    -> alert
    -> response
```

A database can be one storage mechanism, but it is not the definition of logging.

## Logging, monitoring, alerting, and response

My first definitions were too short:

- logging meant creating the record,
- monitoring meant watching events,
- alerting meant notifying about a potentially dangerous event,
- incident response meant acting after detection.

The correction was that monitoring also includes collection, analysis, and correlation. Alerting needs a defined condition, context, owner, and expected action. Incident response includes triage, containment, remediation, recovery, evidence preservation, and learning.

## Authentication logging exercise

I correctly selected the Auth Service as the authoritative source for login and MFA decisions.

My first event list was too broad:

```text
authn_{login/mfa}_{failed/success/recovery/reset}
```

The improved approach uses explicit events:

```text
authn_login_fail
authn_login_success
authn_mfa_fail
authn_mfa_success
session_created
```

Recovery or reset events should be emitted only when that specific workflow occurs.

I also learned that authentication review needs more than login events. Session creation, renewal, invalidation, password-reset token validation, and use of expired or revoked sessions can be security-relevant.

## Detection gap

I correctly stated that recording failed logins does not prove brute-force detection.

The important gap is:

> Events may exist, but no rule correlates them, no alert is delivered, or no owner responds.

The reviewed patterns were:

- many failures against one account,
- one source trying many accounts,
- distributed attempts against one account,
- successful login after repeated failures,
- repeated MFA failures.

I also corrected an unsafe example that placed a password in a login URL. Credentials should not be placed in query strings because URLs can be copied into browser history, proxies, access logs, analytics, and referrer data.

## Log injection exercise

I recognised the main issue:

```text
user-controlled username
    -> interpolated log line
    -> possible manipulation of log data
```

My wording said an attacker could "overwrite" a log. The more precise result in the reviewed example is that an attacker could inject or forge an additional-looking record, corrupt the structure, or mislead a parser. Overwriting existing records would normally require additional access or another weakness.

I also learned to separate:

- the mechanism: manipulation of the log entry,
- the consequence: false interpretation by a parser, SIEM, alert rule, or investigator.

Possible effects include both false negatives and false positives.

## Sensitive logging exercise

I correctly classified the complete request body and session cookie as values that should not be logged.

I initially classified:

```text
actorUserId -> do not log
targetUserId -> do not log
```

That was incorrect for a role-change audit event. These internal identifiers are essential evidence of who changed whose permissions.

I also classified the access token as "masked or transformed" because I thought a hash might help validate CSRF or another control. The correction was:

- do not log the access token,
- do not log a CSRF token,
- log the result of the control, such as `csrf_validation_fail`,
- correlate using a separate request, interaction, or server-generated reference.

Hashing is not an automatic safety control for secrets.

## Minimal role-change audit event

```ts
const auditEvent = {
  timestamp: new Date().toISOString(),
  eventName: "privilege_permissions_changed",
  actorUserId: authenticatedUser.id,
  targetUserId: request.body.userId,
  previousRole,
  newRole: request.body.role,
  result: "success",
  requestId: request.id,
};
```

The main evidence is:

```text
who + target + previous state + new state + time + result
```

Source IP and user agent can support investigation but do not prove identity.

## Severity correction

I initially described severity mainly as impact such as low, medium, high, or critical.

The improved understanding is that severity expresses importance and response priority in context. It can depend on:

- impact,
- asset sensitivity,
- event result,
- confidence,
- frequency,
- account privilege,
- required response speed.

`eventName`, `result`, and `severity` remain separate fields.

## Final takeaway

> Security logging is effective only when the application emits the right safe event, the record remains trustworthy and available, suspicious patterns are detected, an actionable alert reaches an owner, and the response can be executed and verified.
