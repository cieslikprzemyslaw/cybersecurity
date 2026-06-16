# A09 Review Checklist

Use the sections relevant to the reviewed feature. Do not add events without a clear security purpose.

## 1. Asset and action

- What identity, permission, data, transaction, or workflow matters?
- Which actions are high value, privileged, irreversible, or sensitive?
- What abuse case should become visible?

## 2. Security-relevant events

- Are important successes and failures logged?
- Are authentication, MFA, session, and recovery events covered?
- Are authorization denials and privileged actions covered?
- Are role and permission changes auditable?
- Are sensitive create/read/update/delete/export actions covered where justified?
- Are input-validation, token-validation, upload-scan, and rate-limit failures covered?
- Are workflow bypass and out-of-sequence actions visible?

## 3. Event context

- Does the event answer when, where, who, what, and result?
- Is there a stable internal actor identifier?
- Is the target resource or user identified?
- Are `eventName`, `result`, and `severity` separate?
- Is a safe reason code used instead of uncontrolled error text?
- Is there a request, correlation, or interaction ID?
- Are timestamp formats and UTC offsets consistent?
- Are field names, data types, and lengths documented?

## 4. Sensitive-data review

- Are passwords excluded?
- Are session cookies and session IDs excluded?
- Are access, refresh, reset, and MFA tokens excluded?
- Are API keys and cryptographic keys excluded?
- Are `Authorization` headers excluded?
- Are complete request and response bodies avoided?
- Are PII, health, payment, and CMS data minimised?
- Are third-party telemetry and retention settings reviewed?
- Is any masking, hashing, or pseudonymisation actually necessary and safe?

## 5. Event ownership

- Which component enforces the security control?
- Does that component emit the authoritative event?
- Are browser, WAF, database, and infrastructure logs treated as supporting sources where appropriate?
- Can a user suppress or fabricate the only available evidence?

## 6. Log injection

- Which fields are user-controlled?
- Is structured logging used?
- Can newline or control characters change the event structure?
- Can the user control event name, severity, result, or reason code?
- Are field lengths and character handling defined?
- Are downstream parsers and viewers tested?

## 7. Collection and protection

- Where are logs collected?
- Are security logs centralised or reliably forwarded?
- Are logs protected in transit and at rest?
- Who can read, modify, or delete them?
- Are write permissions separated from administrative read access?
- Can logging stoppage, tampering, or deletion be detected?
- Is retention sufficient for delayed investigation and limited by privacy requirements?
- Are backup and disposal processes defined?

## 8. Detection logic

- What exact attack or abuse pattern should be detected?
- Which events and correlation keys are required?
- What threshold and time window are appropriate?
- Could the rule generate obvious false positives?
- Could low-and-slow or distributed behaviour evade it?
- Does a successful action after repeated failures receive higher attention?
- Are DAST and controlled security tests expected to trigger the rule?

## 9. Alerting

- Is the alert actionable and contextual?
- Does it have severity and confidence?
- Is there a named owner or on-call route?
- Is the escalation path documented?
- Is deduplication or suppression used to avoid alert fatigue?
- Is alert delivery monitored?

## 10. Response

- What should happen after a high-confidence alert?
- Are evidence preservation, containment, session invalidation, and account protection considered?
- Is the response proportionate to confidence?
- Does a playbook exist and is it current?
- Are prevention, detection, response, and evidence preservation treated separately?

## 11. Failure handling

- Does failure of the logger, collector, or alert channel fail the security control open?
- Can logging failure exhaust resources or block legitimate traffic?
- Is loss of storage, permissions, network connectivity, or ingestion handled safely?
- Is there monitoring for dropped or delayed events?

## 12. Verification evidence

- Can a test prove the event was emitted?
- Can it prove secrets were excluded?
- Can it prove collection and integrity controls worked?
- Can it prove the detection rule matched?
- Can it prove the alert reached the correct owner?
- Can it prove the response playbook is usable?
