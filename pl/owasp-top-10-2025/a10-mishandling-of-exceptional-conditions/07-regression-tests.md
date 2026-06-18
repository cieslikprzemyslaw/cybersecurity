# A10 Regression Tests

Dobieraj testy do konkretnej funkcji.

## Validation failures

- Missing required field returns `400`.
- Empty required value returns `400`.
- `null` required value returns `400`.
- Array instead of string returns `400`.
- Object instead of string returns `400`.
- Unknown enum returns `400`.
- Malformed identifier returns `400`.
- Oversized body is rejected safely.

## Access and not found

- Valid-looking missing resource returns `404`.
- Unauthenticated request returns `401`.
- Authenticated but unauthorized request returns `403` or controlled `404`.
- Authorization dependency failure causes sensitive action to fail closed.

## Error response safety

- Unexpected repository exception returns controlled `500`.
- Response does not include stack trace, SQL query, filesystem path, secret, token, hostname or config.
- Response includes correlation ID where appropriate.

## Partial state and rollback

- Simulated database failure does not leave active partial records.
- Simulated file-storage failure does not leave completed database state.
- Failed report finalization does not mark report final without immutable snapshot.
- Cleanup failure is logged and monitored.

## Timeout and retry

- Client timeout is treated as unknown outcome.
- UI does not show confirmed success after timeout without backend confirmation.
- Frontend refreshes authoritative state after timeout.
- Retrying status update does not create duplicate activity events.
- Retrying upload does not create duplicate active evidence records.

## Duplicate submit and race basics

- Two concurrent completion requests result in one final status.
- Two concurrent completion requests create at most one completion event.
- Disabled frontend button is not the only protection.
- Backend preserves the business invariant under concurrent requests.

## Resource cleanup

- File handles are released on upload error.
- Database connections are released on repository exception.
- Locks are released on failure.
- Temporary files are removed or marked for cleanup.
- Repeated errors do not cause unbounded queue, memory or storage growth.
