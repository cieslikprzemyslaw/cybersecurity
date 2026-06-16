# Monitoring, Alerting, and Response

## A log is not a detection

A single event can be recorded correctly and still remain unnoticed.

```text
recorded event != detected attack
```

Effective detection normally requires:

```text
events + correlation + rule + threshold + time window + context
```

Effective alerting additionally requires:

```text
severity + recipient + ownership + playbook + timely review
```

## Detection-rule design

A useful rule should define:

1. the attack or abuse pattern,
2. the event source,
3. the correlation key,
4. the threshold,
5. the time window,
6. exclusions or classifications,
7. severity and confidence,
8. alert context,
9. owner and expected response.

### Example: repeated login failures

```text
Pattern: authn_login_fail for the same account
Threshold: 5 events
Window: 5 minutes
Initial severity: WARN
Recipient: security team or on-call owner
Response: review source patterns, rate limiting, account risk, and related sessions
```

### Example: successful login after failures

```text
Pattern: authn_login_success after at least 5 authn_login_fail events
Correlation: same account
Window: 10 minutes
Severity: HIGH, or higher for a privileged account
Recipient: security team or on-call owner
Response: review MFA, session, source, and activity after login; invalidate sessions when confidence is high
```

The thresholds are illustrative. They must be tuned to the application's users, traffic, risk, and lockout or rate-limiting design.

## Correlation choices

Useful correlation keys can include:

- account ID,
- target resource,
- request or interaction ID,
- source IP as contextual evidence,
- device or user-agent characteristics,
- application and environment,
- time window.

An IP address is not a reliable user identity. NAT, proxies, mobile networks, VPNs, and shared networks can group unrelated users or split one user across different addresses.

## Actionable alerts

An actionable alert should contain enough context to decide what to do next:

- event and detection-rule name,
- affected account or resource,
- relevant event sequence,
- time range,
- source context,
- confidence and severity,
- current session or account state,
- response guidance,
- link to the supporting evidence,
- named owner or escalation route.

An alert without context creates investigation delay. An alert without an owner becomes unattended work.

## False positives and false negatives

### True positive

The rule correctly identifies suspicious or malicious behaviour.

### False positive

The alert fires for legitimate or non-security-relevant behaviour.

### False negative

Suspicious or malicious behaviour occurs but the rule does not alert.

### Accepted risk

A real security risk is documented and accepted by an authorised owner. This is not the same as a false positive.

## Alert fatigue

Alerting on every individual failure creates noise. High alert volume can cause important events to be reviewed too late or ignored.

Controls include:

- correlation instead of one-alert-per-event,
- risk-based thresholds,
- account and asset context,
- deduplication and suppression windows,
- tuning based on evidence,
- regular review of rules and playbooks.

## Response flow

A high-confidence alert may lead to:

1. triage and validation,
2. evidence preservation,
3. session invalidation or containment,
4. account protection,
5. review of activity after compromise,
6. remediation of the original weakness,
7. recovery and communication,
8. rule and playbook improvement.

Logging and alerting do not replace prevention. They identify attempted or successful abuse and support action after detection.

## Honeytokens

A honeytoken is a decoy value or identity that should not be accessed during normal business activity. Its use can create a high-confidence event with few legitimate explanations.

Honeytokens still require:

- protected placement,
- a reliable event source,
- a tested alert,
- an owner and playbook,
- careful handling to avoid operational harm.

## Verification

A detection is not complete until the full path has been tested:

```text
test action
    -> expected event emitted
    -> event collected
    -> rule matched
    -> alert delivered
    -> alert contains useful context
    -> owner can follow the playbook
```

DAST, penetration testing, and controlled abuse-case tests should create expected security events and alerts. If they do not, the result is evidence of a detection gap.
