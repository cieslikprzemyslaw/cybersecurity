# Privileged Role Changes Lack Sufficient Security Logging and Alerting

## Severity

Medium

The final severity depends on the application's privilege model, data sensitivity, existing administrative controls, and other monitoring sources. It may be higher in a high-risk or regulated environment.

## Category

- OWASP Top 10 2025: A09 Security Logging and Alerting Failures
- Related concerns: accountability, privileged access, incident detection, forensic readiness
- Relevant weakness patterns: omission of security-relevant information and insufficient logging

## Summary

The application permits an administrator to change another user's role, but the reviewed flow does not produce a complete, protected audit event containing the actor, target, previous role, new role, timestamp, and result.

No verified detection rule or alert was available for unexpected or high-risk privilege changes.

As a result, an unauthorised, compromised, or accidental role change may not be detected promptly, and later investigation may be unable to determine who performed the change or reconstruct the affected state.

## Affected flow

```text
administrator request
    -> role-change endpoint
    -> database update
    -> successful response
    -> missing or incomplete audit event
    -> no verified detection or alert
```

## Evidence

In the sanitised review scenario:

- the role-change operation returned a successful result,
- the proposed log contained unsafe fields such as a session cookie, access token, and complete request body,
- actor and target internal identifiers were initially omitted from the minimum audit design,
- no evidence demonstrated a correlation rule, alert owner, or response playbook for suspicious privilege changes.

This is an example finding based on the completed design exercise rather than a claim about a production system.

## Impact

A malicious or compromised administrator account could change privileges with reduced likelihood of timely detection.

Potential consequences include:

- unauthorised administrative access,
- access to sensitive data or functions,
- loss of accountability,
- delayed incident response,
- incomplete forensic evidence,
- regulatory or audit difficulties.

The logging failure does not itself grant the privilege. It reduces visibility and the ability to detect and investigate abuse of the privilege-changing function.

## Root cause

Security-event requirements were not defined as part of the privileged workflow. The implementation focused on completing the database update but did not establish:

- the authoritative event schema,
- sensitive-data exclusions,
- protected audit storage,
- detection logic,
- alert ownership,
- response requirements.

## Security requirement

> Every privileged role or permission change must create a protected audit event containing the actor's internal identifier, target user's internal identifier, previous value, new value, timestamp, result, and correlation identifier.

Additional requirements:

> Authentication secrets, session cookies, access tokens, passwords, and complete request bodies must not be recorded in the audit event.

> Unexpected or high-risk privilege changes must be covered by a documented detection rule, alert owner, and response playbook.

## Recommended remediation

### Event generation

- Emit a structured event from the backend service that enforces and performs the role change.
- Use an application-controlled event name such as `privilege_permissions_changed`.
- Record the actor, target, previous role, new role, timestamp, result, and request ID.
- Record failed and denied role-change attempts where they support detection.

### Sensitive-data controls

- Build the event from an allowlist of approved fields.
- Do not log the complete request body.
- Exclude passwords, tokens, cookies, and authorization headers.
- Use internal IDs rather than unnecessary personal data.

### Protection and detection

- Forward the event to protected central collection.
- Restrict read, write, modification, and deletion permissions.
- Define detections for unexpected administrator assignment, repeated denied changes, use of emergency accounts, and privilege changes outside normal process.
- Assign alert ownership and document the response.

## Regression tests

1. Perform a successful role change and confirm one complete audit event exists.
2. Confirm the event contains actor ID, target ID, previous role, new role, timestamp, result, and request ID.
3. Confirm the event excludes password, cookie, access token, and full request body.
4. Attempt the change without permission and confirm denial plus an appropriate security event.
5. Submit newline and control characters in any loggable justification or username field and confirm one valid structured event.
6. Attempt to modify or delete the stored audit record and confirm the action is denied or detected.
7. Trigger the defined suspicious-role-change condition and confirm the alert reaches the documented owner.

## Developer takeaway

A successful database update is not a sufficient audit trail. Privileged workflows need deliberately designed evidence, protection, detection, ownership, and response.
