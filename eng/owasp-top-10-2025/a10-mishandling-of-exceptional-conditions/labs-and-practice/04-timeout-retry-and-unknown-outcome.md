# Practice 04: Timeout, Retry and Unknown Outcome

## Scenario

The frontend sends:

```http
PATCH /api/assessments/asm_123/status

{
  "status": "completed"
}
```

Normal flow:

```text
1. User clicks Complete Assessment.
2. Backend checks auth/authz.
3. Backend checks the state transition.
4. Backend saves status = completed.
5. Backend creates an activity log entry.
6. Backend returns 200 OK.
7. Frontend shows completed.
```

Failure scenario:

```text
Backend completes the operation, but the frontend times out before receiving the response.
```

The frontend only sees:

```text
Network request failed / timeout
```

## What I corrected

I first thought the frontend could assume the assessment was not completed because no response arrived.

Corrected model:

```text
The frontend can say: success was not confirmed.
The frontend cannot say as a fact: the backend did nothing.
```

A timeout means unknown outcome.

The server may have:

- not received the request,
- received only part of the request,
- received the request but failed before changing state,
- changed state but failed before returning a response,
- completed successfully but the response was lost.

## A10 learning point

Timeout is not proof of failure. Blind retry can duplicate side effects.

Possible duplicated side effects:

- duplicate activity logs,
- repeated emails or notifications,
- duplicate file uploads,
- repeated payment attempts,
- repeated status-change side effects.

## Safe expected behaviour

The frontend should:

- not show confirmed success without confirmation,
- not assume the backend did nothing,
- show an "unable to confirm" or controlled error state,
- refresh the backend state where possible,
- avoid infinite automatic retry loops,
- allow retry only when it is safe or after status is checked.

The backend should:

- handle repeated requests safely,
- avoid duplicating side effects when the final state is already reached,
- return the current state when a repeated operation is already complete,
- preserve the security invariant.

## Regression tests

Useful tests:

- simulate frontend timeout after backend status update,
- assert that UI does not show confirmed success without confirmation,
- assert that refreshing the assessment shows the real backend state,
- retrying the same status change does not duplicate the completion activity log,
- repeated requests return a safe current state or controlled rejection,
- timeout does not leave optimistic UI permanently showing an unconfirmed sensitive action.

## Frontend takeaway

The correct UI state after timeout is not "success" and not necessarily "nothing happened". It is "unknown / unable to confirm" until the backend state is checked.
