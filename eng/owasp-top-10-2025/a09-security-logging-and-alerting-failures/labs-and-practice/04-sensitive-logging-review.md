# Sensitive Logging Review

## Scenario

A backend logs a successful administrator role change with fields including actor details, target details, source context, tokens, request body, old and new roles, and result.

## Final classification

| Field | Classification | Reasoning |
|---|---|---|
| `timestamp` | Log directly | Required for event ordering and investigation. |
| `eventName` | Log directly | Defines the security event. |
| `actorUsername` | Mask, transform, or controlled use | May be PII and can change; internal ID is preferred. |
| `actorUserId` | Log directly | Essential evidence of who performed the action. |
| `targetUserId` | Log directly | Essential evidence of whose role changed. |
| `sourceIp` | Direct or transformed by policy | Useful context, not reliable identity. |
| `userAgent` | Controlled use | Useful for some investigations but not always necessary. |
| `sessionCookie` | Do not log | Authentication secret. |
| `accessToken` | Do not log | Authentication secret; hashing is not required for this event. |
| `requestId` | Log directly | Safe correlation across services. |
| `errorCode` | Log directly when safe | Use an application-controlled reason code, not a stack trace. |
| `requestBody` | Do not log | Can contain passwords, tokens, justification text, and unnecessary data. |
| `previousRole` | Log directly | Required to reconstruct the change. |
| `newRole` | Log directly | Required to reconstruct the change. |
| `result` | Log directly | Shows whether the action succeeded. |

## Minimal audit evidence

```text
actorUserId
targetUserId
previousRole
newRole
timestamp
result
```

`eventName` and `requestId` make the record clearer and easier to correlate.

## Safe event

```ts
logger.warn("User role changed", {
  timestamp: new Date().toISOString(),
  eventName: "privilege_permissions_changed",
  actorUserId: authenticatedUser.id,
  targetUserId: request.body.userId,
  previousRole,
  newRole: request.body.role,
  result: "success",
  requestId: request.id,
});
```

## CSRF clarification

A CSRF token should not be logged to prove whether validation worked. Log the control result instead:

```json
{
  "eventName": "csrf_validation_fail",
  "actorUserId": "usr_123",
  "requestId": "req_456",
  "route": "/api/users/usr_987/role",
  "method": "PATCH",
  "result": "failure",
  "reasonCode": "TOKEN_MISSING_OR_INVALID"
}
```

## Learning correction

I initially removed actor and target IDs while considering an access-token hash acceptable. The correction was that internal actor and target IDs are necessary audit evidence, while the access token itself is not needed and should not be recorded.
