# Sensitive Data and Log Injection

## Logs are security assets and attack surfaces

Logs support defence, but they can also be targeted.

### Confidentiality

Unauthorised readers may obtain:

- personal or health data,
- passwords or tokens,
- internal paths and hostnames,
- database connection details,
- commercially sensitive information,
- security-control behaviour.

### Integrity

Attackers may:

- forge entries,
- alter fields,
- inject additional lines,
- delete evidence,
- change the recorded actor,
- corrupt a parser or downstream viewer.

### Availability

Attackers may:

- flood logs,
- exhaust disk or ingestion quotas,
- prevent future events from being recorded,
- overload collectors or analytics,
- exploit expensive synchronous logging paths to degrade the application.

### Accountability

Attackers may suppress writes, damage evidence, or cause the wrong identity to be recorded so responsibility cannot be assigned reliably.

## Data that should normally not be logged

- plaintext passwords,
- session cookies or session identifiers,
- access or refresh tokens,
- password-reset tokens,
- MFA secrets, codes, or recovery codes,
- API keys,
- private cryptographic keys,
- complete `Authorization` headers,
- database connection strings,
- full payment-card data,
- unnecessary sensitive personal or health data,
- complete request or response bodies without a reviewed requirement,
- environment-variable dumps,
- confidential CMS content,
- source code.

## Values that may require controlled handling

Depending on the investigation need and legal basis:

- usernames and email addresses,
- source IP addresses,
- user agents,
- internal hostnames,
- file paths,
- limited error details,
- masked account references.

Possible controls include:

- internal IDs instead of display names,
- masking or pseudonymisation,
- restricted exceptional logs,
- shorter retention,
- separate protected storage,
- role-based access and auditing of log access.

Hashing does not automatically make a secret safe. A hash can remain a stable identifier, may support guessing attacks for predictable values, and may be unnecessary when an independent correlation ID can be used.

## Correlation without logging authentication secrets

Prefer:

```text
requestId
correlationId
interactionId
server-generated session reference
```

Do not use the real access token or session cookie as the correlation value.

## Log injection mental model

```text
untrusted input
    -> log message
    -> parser / viewer / SIEM
    -> misleading or dangerous interpretation
```

Potentially controlled fields include:

- username,
- filename,
- query parameter,
- header,
- form value,
- user-agent string,
- error text from another system.

Newline and control characters can:

- create a fake second entry,
- hide a real event,
- break JSON or line-oriented parsing,
- assign the wrong severity or actor,
- create false positives or false negatives,
- mislead an investigator.

## Unsafe pattern

```ts
logger.warn(
  `${new Date().toISOString()} LOGIN_FAILED username=${username} ip=${request.ip}`
);
```

The user-controlled value is inserted into the structure of a manually composed line.

## Safer pattern

```ts
logger.warn("Authentication failed", {
  timestamp: new Date().toISOString(),
  eventName: "authn_login_fail",
  accountId,
  sourceIp: request.ip,
  requestId: request.id,
  result: "failure",
  reasonCode: "INVALID_CREDENTIALS",
});
```

Security properties:

- the application controls `eventName`, severity, result, and reason code,
- user input remains a field value,
- the logger performs structured serialisation,
- field length and control characters can be validated or normalised,
- the password is excluded.

Structured logging reduces risk but is not magic. The chosen library, transport, parser, viewer, and downstream integrations must preserve the structure safely.

## Primary controls

- use structured logging,
- avoid string concatenation with untrusted values,
- encode, escape, reject, or normalise unsafe control characters as appropriate,
- define maximum field lengths,
- use safe application-controlled event names and reason codes,
- test downstream parsing and display,
- restrict access to logs,
- protect stored records from alteration and deletion.

## Regression expectation

A test value containing newline or control characters should produce exactly one valid event. It must not change:

- event type,
- severity,
- result,
- reason code,
- field boundaries,
- number of recorded events.
