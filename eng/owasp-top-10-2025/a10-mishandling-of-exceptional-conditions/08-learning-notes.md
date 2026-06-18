# A10 Learning Notes

## Initial understanding

I first described A10 as not considering exceptions we did not predict. That was close, but too narrow.

Better model:

```text
The issue is not that an error happened.
The issue is unsafe behaviour after an abnormal condition.
```

A condition may be anticipated as a possible failure type, even if the exact runtime cause is unknown. Examples include dependency timeout, database failure, missing state, malformed data, failed cleanup, concurrent requests and resource exhaustion.

## Key clarifications

### Generic 500 is only the client response

A generic response reduces information disclosure, but it does not prove rollback, cleanup or safe recovery.

The response tells me what the client received. It does not tell me what happened to database rows, files, emails, logs, queues, sessions or third-party side effects.

### Timeout means unknown outcome

A browser timeout means the frontend did not receive confirmation. The backend may have received and processed the request.

Safe UI state:

```text
unable to confirm
```

not:

```text
nothing happened
```

### Frontend controls are UX controls

Frontend validation, disabled buttons and React error boundaries help users, but they do not enforce backend security.

They can reduce accidental mistakes, but they do not stop Burp, ZAP, Postman, DevTools, scripts, multiple tabs or direct API requests.

### A01 / A06 / A10 distinction

```text
No authorization check exists -> A01 / A06
Authorization check exists but fails open during dependency failure -> A10
```

Missing auth/authz is usually not a `500` and not automatically A10. It becomes A10 when an existing control reaches an exceptional condition and the application responds insecurely.

## Practice completed

### API error handling

I reviewed validation, not-found, access and unexpected exception cases.

Learning:

```text
400 = malformed or invalid request
401 = unauthenticated
403 = authenticated but not allowed
404 = not found or intentionally hidden
500 = unexpected server-side or dependency failure
```

### Partial state

Scenario:

```text
create evidence record -> save file -> update assessment count -> failure
```

Learning:

```text
500 does not prove nothing changed.
Backend should not leave active inconsistent state.
Regression tests should prove rollback or cleanup.
```

### Missing and malformed parameters

Scenario:

```text
PATCH assessment status with {}, "", null, ["completed"], "admin" and "completed"
```

Learning:

```text
Bad input shape should be rejected as controlled validation failure.
A valid input value still needs authentication, authorization and allowed state transition checks.
```

### Timeout and retry

Learning:

```text
timeout = unknown outcome
retry may duplicate side effects
frontend should refresh backend state
backend should handle repeated requests safely
```

### Duplicate submit / race basics

Learning:

```text
disabled button is not enough
backend must preserve the invariant
two concurrent complete requests must not create two completion events
```

## My practical review checklist

When I review a flow, I ask:

1. What is the normal flow?
2. What abnormal condition can occur?
3. What state could already be changed?
4. Does the system fail open or fail closed?
5. Is there partial state?
6. Is rollback, cleanup or recovery needed?
7. What should the user see?
8. What should be logged internally?
9. Could retry duplicate a side effect?
10. What regression test proves the safe state?

## Interview-style explanation

> A10 is about unsafe application behaviour when an abnormal condition occurs. I would not treat every exception as a vulnerability. I would check what condition occurred, what state had already changed, whether the system failed open or closed, whether partial state remained, whether retry could duplicate side effects, whether the client saw sensitive internal details, and which regression test proves the system returned to a known safe state.
