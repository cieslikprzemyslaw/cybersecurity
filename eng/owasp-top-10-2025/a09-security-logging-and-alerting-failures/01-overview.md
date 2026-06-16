# A09 Overview

## Definition

Security Logging and Alerting Failures occur when an application or organisation cannot create, preserve, analyse, detect, escalate, or respond to security-relevant events effectively.

Typical failures include:

- important events are not logged,
- events are logged inconsistently or without useful context,
- secrets or unnecessary personal data are written to logs,
- untrusted data can forge or corrupt log entries,
- logs can be modified, deleted, or accessed by unauthorised users,
- events are stored only locally and are unavailable during an investigation,
- suspicious patterns are not monitored or correlated,
- alerts have no useful threshold, context, owner, or playbook,
- excessive false positives hide real attacks,
- logging or alerting failures cause a security control to fail open.

OWASP A09:2025 remains at number nine and gives alerting more emphasis because recorded events only support active defence when relevant patterns cause timely action.

## Important distinctions

### Application log

A record produced by the application about behaviour, state, errors, or operations.

### Security log

A record relevant to detection, accountability, investigation, or protection of a security control.

### Audit log

A chronological record of a sensitive action, commonly including the actor, target, change, time, and result.

### Debug log

Detailed troubleshooting information. Debug output may contain stack traces, internal paths, configuration, request data, or secrets and should not be enabled casually in production.

### Monitoring

Collection, review, analysis, and correlation of logs, metrics, and events over time.

### Alerting

A notification or automated action generated when a defined detection condition is met.

### Incident response

Investigation, triage, containment, remediation, recovery, evidence preservation, and learning after an incident is detected.

## A09 is not only "missing logs"

The category includes the complete defensive chain:

```text
required event
    -> correct and safe event structure
    -> trustworthy storage
    -> monitoring and correlation
    -> actionable alert
    -> owned response
```

Any broken step can create a visibility, detection, or response gap.

## Facts, assumptions, and gaps

A logging review should separate:

### Facts

- which event was actually emitted,
- which fields were present,
- where it was stored,
- whether a rule matched,
- whether an alert was delivered,
- who received it,
- which response action occurred.

### Assumptions

- "the platform probably logs this",
- "the security team probably monitors it",
- "the alert must have been delivered",
- "the IP address proves who performed the action".

### Detection gap

A malicious or suspicious pattern can occur without a useful rule identifying it.

### Response gap

An alert exists, but no owner, escalation path, or playbook causes timely action.

## Relevant weakness patterns

A09 includes several related weakness classes, especially:

- improper output neutralisation for logs,
- omission of security-relevant information,
- sensitive information written to logs,
- insufficient logging.

## Impact

The impact is often indirect but serious:

- account takeover or privilege abuse remains unnoticed,
- attackers can operate for longer,
- incident scope cannot be reconstructed,
- evidence is incomplete or untrustworthy,
- alerts are delayed or ignored,
- personal data or secrets leak through logging systems,
- operational and regulatory impact increases.

Logging does not prevent the original vulnerability. It supports visibility, detection, accountability, investigation, and response.

## Interview-style explanation

> A09 is the failure of the security-event lifecycle. The application may omit an event, record unsafe or incomplete data, fail to protect the log, or record the event without any useful detection and alerting. As a frontend developer moving into AppSec, I would verify where the authoritative backend decision is made, which structured event is emitted, which secrets are excluded, what rule detects abuse, who owns the alert, and how the complete flow is tested.
