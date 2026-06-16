# Frontend and API Logging

## Appropriate role of frontend telemetry

Frontend logging is not useless. It can support:

- JavaScript error reporting,
- failed resource or API-call diagnostics,
- performance monitoring,
- UX and browser compatibility investigation,
- correlation with backend requests,
- Content Security Policy reports,
- identification of unusual client-side behaviour.

However, the browser is controlled by the user. Client telemetry can be modified, blocked, replayed, or fabricated.

## Authoritative event ownership

The component that enforces the control should normally emit the authoritative security event.

| Security decision | Primary authoritative source |
|---|---|
| Password accepted or rejected | Identity or Auth Service |
| MFA completed or failed | Identity or Auth Service |
| Session created or invalidated | Session or Auth Service |
| Authorization allowed or denied | API or service enforcing access |
| Role changed | Administrative backend service |
| File accepted, rejected, or quarantined | Upload validation or malware-scanning service |
| Sensitive data exported | Business service performing the export |
| Rate limit applied | Gateway, middleware, or service enforcing the limit |

Supporting sources may include:

- WAF,
- API gateway,
- web server,
- database audit logs,
- platform logs,
- frontend error telemetry.

Supporting evidence should not replace the application event when the application has the best knowledge of actor, target, action, and result.

## Do not expose secrets in browser logging

Avoid patterns such as:

```ts
console.log("token", accessToken);
console.log("login payload", formValues);
console.log("API response", responseBody);
```

Risks include:

- local access to developer tools,
- browser-extension access,
- screen sharing or support capture,
- third-party telemetry ingestion,
- excessive retention,
- access by too many staff members.

## Error tracking and data minimisation

Before sending an event to a frontend error-tracking service, review:

- URL and query string,
- request headers,
- cookies,
- form fields,
- DOM snapshots,
- API response data,
- user identifiers,
- CMS content,
- source maps and stack traces.

Use allowlists, redaction hooks, sampling, retention limits, and access controls. Do not rely only on a provider's default scrubbing.

## Correlation pattern

A safe request flow may use a server-generated request ID:

```text
browser request with request ID
    -> API receives request ID or creates a trusted one
    -> backend emits authoritative security event
    -> frontend error telemetry records the same non-secret ID
    -> monitoring correlates both sources
```

The request ID supports investigation but does not grant authentication or authorization.

## API event example

```ts
logger.warn("Authorization denied", {
  timestamp: new Date().toISOString(),
  eventName: "authz_fail",
  actorUserId: request.auth.userId,
  affectedResource: `company:${request.params.companyId}`,
  action: "company.update",
  result: "failure",
  reasonCode: "INSUFFICIENT_PERMISSION",
  requestId: request.id,
  sourceIp: request.ip,
});
```

Do not include:

- the access token,
- complete cookies,
- full request body,
- sensitive resource contents,
- stack traces in the normal security event.

## Production error handling

The user-facing response and internal event have different purposes.

User response:

```json
{
  "message": "The request could not be completed."
}
```

Internal event:

```json
{
  "eventName": "security_control_error",
  "requestId": "req_123",
  "service": "permissions-api",
  "result": "failure",
  "reasonCode": "POLICY_ENGINE_UNAVAILABLE"
}
```

A generic user response does not justify an empty internal log. A detailed internal event does not justify leaking implementation details to the user.
