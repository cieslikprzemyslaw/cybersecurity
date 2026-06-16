# A09 Practical Summary

## What I reviewed

- application-level security events,
- authentication and MFA event naming,
- correlation and alert thresholds,
- log injection in a string-built Node/Express log line,
- safe fields for a privileged role-change audit event,
- frontend telemetry versus authoritative backend evidence.

## Main detection gap

The clearest gap was not only missing logs. It was the situation where events were present but no rule correlated them, no alert reached an owner, or no response process followed.

## Main sensitive-data lesson

The investigation needs evidence of actor, target, action, time, and result. It does not need passwords, access tokens, session cookies, MFA codes, or complete request bodies.

## Main log-injection lesson

String interpolation allowed a controlled value to influence the log line's structure. The safer design uses structured events, application-controlled metadata, field limits, and regression tests against newline and control characters.

## Security requirements produced

- Authentication successes and failures must create structured security events.
- Privileged role changes must create an audit event containing actor, target, previous role, new role, timestamp, and result.
- Logs must not contain authentication secrets.
- User-controlled values must not control event structure, name, severity, result, or reason code.
- Repeated authentication failures and success-after-failure patterns must have defined detection rules.
- High-severity alerts must have an owner and response playbook.
- Security logs must be protected from unauthorised access, modification, and deletion.

## Final assessment

**PASS — Frontend Developer transitioning into AppSec**

I can review a simple feature and ask:

```text
what happened?
what was logged?
what was excluded?
what pattern should alert?
who owns it?
what evidence proves the flow works?
```
