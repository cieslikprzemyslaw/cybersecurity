# TryHackMe A09 Practice

## Source

TryHackMe - OWASP Top 10 2025: IAAA Failures, A09 section.

## Scope

I completed the A09 material after already finishing the A01 and A07 parts of the room. I used the room together with the OWASP A09 page and both OWASP logging cheat sheets.

The retained learning was not a claim that one specific commercial monitoring stack is required. The application must first emit useful security events, and the wider system must then collect, monitor, correlate, alert, and support response.

## Main observations

- Missing logs make attacks difficult or impossible to reconstruct.
- Logs without monitoring and alerting may remain unnoticed during an active attack.
- Important successes and failures both need consideration.
- Logging sensitive data creates a new confidentiality risk.
- Untrusted values can attack the logging pipeline itself.
- Local-only logs can be unavailable, deleted, or difficult to correlate.
- Excessive false positives can hide important alerts.
- A detection without a current playbook creates a response gap.

## Evidence standard

For A09, useful evidence would include:

```text
expected security action
    -> emitted event
    -> collected event
    -> matching detection rule
    -> delivered alert
    -> documented response
```

I do not claim that an attack was detected merely because one event existed.

## Result

The TryHackMe section provided the category context. The practical understanding was consolidated through the authentication plan, log-injection review, and sensitive-data classification exercises documented in this folder.
