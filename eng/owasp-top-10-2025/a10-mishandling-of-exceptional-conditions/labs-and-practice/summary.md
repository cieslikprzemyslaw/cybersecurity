# A10 Practice Summary

## Completed exercises

1. [API error-handling review](01-api-error-handling-review.md)
2. [Transaction and rollback review](02-transaction-and-rollback-review.md)
3. [Missing and malformed parameters](03-missing-and-malformed-parameters.md)
4. [Timeout, retry and unknown outcome](04-timeout-retry-and-unknown-outcome.md)
5. [Duplicate submit and race-condition basics](05-duplicate-submit-and-race-basics.md)

## Final status

PASS — Frontend Developer transitioning into AppSec.

## Main practical model

```text
abnormal condition
-> what changed already?
-> did the application fail open or fail closed?
-> is there partial state?
-> is retry safe?
-> what should the frontend show?
-> what must the backend enforce?
-> what regression test proves the safe state?
```

## What I can explain

- A10 is unsafe behaviour during abnormal conditions, not every exception.
- Expected validation failures should produce controlled `4xx` responses.
- Unexpected server or dependency failures may produce controlled `5xx` responses.
- Fail-open means continuing when a control cannot make a safe decision.
- Fail-closed means preserving the security rule and safe state.
- Generic error responses reduce information disclosure but do not prove recovery.
- `try/catch` and logging alone do not prove safe handling.
- Timeout means unknown outcome.
- Retry may duplicate side effects.
- Frontend validation and disabled buttons are UX controls, not backend security controls.
- Missing auth/authz is usually A01/A06; unsafe behaviour when an existing control fails can be A10.

## Exercise evidence

### 1. API error handling

I reviewed how to separate validation, access, not-found and unexpected failure cases.

Key takeaway:

```text
400 = malformed or invalid request
401 = unauthenticated
403 = authenticated but not allowed
404 = not found or intentionally hidden
500 = unexpected server-side or dependency failure
```

### 2. Transaction and rollback

I reviewed a multi-step evidence upload flow.

Key takeaway:

```text
500 does not prove that nothing changed.
Partial state can remain if one step succeeds and a later step fails.
```

### 3. Missing and malformed parameters

I reviewed missing, empty, null, array and unknown enum values.

Key takeaway:

```text
Unexpected input shapes should be rejected as controlled validation failures before they reach business logic.
```

### 4. Timeout and retry

I reviewed a frontend timeout after a backend status update.

Key takeaway:

```text
timeout = unknown outcome
unknown outcome should not be blindly retried or shown as confirmed success
```

### 5. Duplicate submit / race basics

I reviewed two concurrent `Complete Assessment` requests.

Key takeaway:

```text
disabled button is UX, not security
backend must preserve the invariant: one completion -> one completion event
```

## Interview-style explanation

> A10 is about unsafe application behaviour when an abnormal condition occurs. I would not treat every exception as a vulnerability. I would check what condition occurred, what state had already changed, whether the system failed open or closed, whether partial state remained, whether retry could duplicate side effects, whether the client saw sensitive internal details, and which regression test proves the system returned to a known safe state.
