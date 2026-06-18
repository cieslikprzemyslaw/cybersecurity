# Practice 05: Duplicate Submit and Race-Condition Basics

## Scenario

Two requests are sent almost at the same time:

```http
PATCH /api/assessments/asm_123/status

{
  "status": "completed"
}
```

Business invariant:

```text
An assessment can be completed once.
One completion should create one completion event.
```

Problem flow:

```text
Request A reads status = in-progress.
Request B reads status = in-progress.
Request A updates status to completed and creates an activity event.
Request B also updates status to completed and creates an activity event.
```

Final state:

```text
assessment.status = completed
activity log contains two "Assessment completed" entries
```

## Shared state

The shared state is not only the ID. The ID identifies the shared resource.

Shared state includes:

- `assessment.status`,
- completion timestamp,
- activity log entries for that assessment,
- any counters or report state affected by completion.

## What I understood

Frontend disabled buttons reduce accidental double-clicks, but they do not enforce security.

Requests can still be sent using:

- Burp Suite,
- OWASP ZAP,
- Postman,
- DevTools,
- scripts,
- another browser tab,
- slow network retries,
- or mobile/browser retry behaviour.

## A10 learning point

This can be treated as A10 when the application mishandles an abnormal timing/state condition:

```text
two concurrent requests -> same mutable state -> check-before-update race -> duplicate side effect
```

It may also be described as a business logic bug. It becomes security-relevant when it breaks integrity, access control, auditability, billing, workflow rules or other security invariants.

## Safe expected behaviour

The backend should preserve the invariant even when concurrent requests arrive.

At my current level, I do not need to design the full locking strategy. I need to recognise the requirement:

```text
Two concurrent completion requests must not create two completion side effects.
```

Safe outcomes may include:

- one request completes and the second returns the current completed state,
- one request completes and the second is safely rejected,
- both requests return a safe result, but only one completion event exists.

## Regression tests

Useful tests:

- send two concurrent `PATCH completed` requests for the same assessment,
- assert the assessment ends as `completed`,
- assert only one completion activity event exists,
- assert no duplicate notification/email/report-finalisation side effect occurs,
- assert the UI remains consistent after refresh,
- assert frontend disabled button is treated as UX only, not the only control.

## Frontend takeaway

The frontend should prevent accidental duplicate clicks for UX, but the backend must preserve the workflow invariant. If the backend relies only on the disabled button, the control is bypassable.
