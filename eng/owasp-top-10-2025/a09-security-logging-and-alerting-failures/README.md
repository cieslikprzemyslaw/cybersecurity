# A09: Security Logging and Alerting Failures

This folder contains my practical notes for OWASP Top 10 2025 - A09: Security Logging and Alerting Failures.

## Status

**PASS — Frontend Developer transitioning into AppSec**

Completed work:

- TryHackMe: OWASP Top 10 2025 - IAAA Failures, A09 section,
- OWASP A09:2025 review,
- OWASP Logging Cheat Sheet review,
- OWASP Logging Vocabulary Cheat Sheet review,
- authentication logging and alerting plan,
- log injection code review,
- sensitive logging classification exercise,
- example security finding and regression-test design.

The status does not mean expert-level SOC, SIEM, incident response, or security-architecture knowledge. It means I can identify useful security events, recognise unsafe logging, separate logging from monitoring and alerting, review a simple detection flow, and propose developer-focused controls and regression tests.

## Core mental model

```text
security event
    -> structured log
    -> protected collection
    -> correlation and detection
    -> alert
    -> investigation and response
```

A log is evidence that an event was recorded. It is not proof that the event was detected, reviewed, or acted upon.

## Architecture overview

[![A09 security logging and alerting architecture](assets/security-logging-architecture.png)](assets/security-logging-architecture.png)

The diagram separates:

- browser and frontend telemetry,
- authoritative application and authentication events,
- supporting database and platform logs,
- collection and protected storage,
- detection, alerting, ownership, and response.

## Main review questions

1. What security-relevant event occurred?
2. Which component made the authoritative security decision?
3. What context is required to investigate it?
4. Which secrets or sensitive values must be excluded?
5. Can untrusted input alter the structure or meaning of the log?
6. Where is the event collected and protected?
7. What sequence or threshold should trigger detection?
8. Who owns the alert?
9. What response should follow?
10. What evidence proves the complete flow works?

## Frontend perspective

Browser logs are useful, but they are not authoritative security audit records.

- Users can modify, suppress, or fabricate client-side logs.
- Frontend telemetry can support error diagnosis, performance monitoring, and correlation.
- Authentication and authorization results must be recorded where the backend enforces them.
- Passwords, session cookies, access tokens, reset tokens, API keys, and sensitive response bodies must not be written to the browser console or telemetry tools.
- Correlation IDs can connect frontend and backend events without exposing real authentication secrets.

## Start here

- [Overview](01-overview.md)
- [Security events and audit logs](02-security-events-and-audit-logs.md)
- [Monitoring, alerting, and response](03-monitoring-alerting-and-response.md)
- [Sensitive data and log injection](04-sensitive-data-and-log-injection.md)
- [Frontend and API logging](05-frontend-and-api-logging.md)
- [Review checklist](06-checklist.md)
- [Regression tests](07-regression-tests.md)
- [Learning notes](08-learning-notes.md)
- [Labs and practice](labs-and-practice/README.md)
- [Example security finding](security-findings/01-example-finding-insufficient-security-logging.md)

## Direct learning links

- [OWASP A09:2025 - Security Logging and Alerting Failures](https://owasp.org/Top10/2025/A09_2025-Security_Logging_and_Alerting_Failures/)
- [TryHackMe - OWASP Top 10 2025: IAAA Failures](https://tryhackme.com/room/owasptopten2025one)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [OWASP Logging Vocabulary Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Vocabulary_Cheat_Sheet.html)
