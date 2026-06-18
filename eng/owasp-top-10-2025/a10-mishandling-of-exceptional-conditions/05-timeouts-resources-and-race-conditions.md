# Timeouts, Resources and Race Conditions

## Timeout means unknown outcome

A timeout means the client did not receive a response in time.

It does not prove:

- the backend did not receive the request,
- the backend did not process the request,
- rollback happened,
- retry is safe.

Safer frontend wording:

```text
The operation could not be confirmed.
```

not:

```text
The operation definitely failed and nothing happened.
```

## Dependency failure

Review whether fallback is less secure than the normal path.

Unsafe examples:

- cannot verify authorization -> allow request,
- cannot scan uploaded file -> accept as safe,
- cannot validate token -> treat as logged in,
- cannot record audit event -> perform sensitive action with no accountability.

## Resource exhaustion

Exceptional conditions can create denial-of-service risk when resources are not limited or released:

- file handles left open,
- database connections not released,
- locks not cleared,
- temporary files not deleted,
- repeated retries overload dependencies,
- huge request bodies are accepted,
- error paths are expensive.

Useful controls include request-size limits, rate limits, timeouts, quotas, concurrency limits, bounded queues, cleanup in `finally`, and monitoring repeated failures.

## Race condition basics

Core model:

```text
two requests
    -> same mutable state
    -> check before update
    -> both pass the check
    -> invalid outcome
```

Example:

```text
Request A reads status = in-progress
Request B reads status = in-progress
A sets completed and creates activity event
B sets completed and creates activity event
```

Final status may be correct, but duplicate side effects exist.

## Disabled button is not enough

A disabled button helps UX, but it does not stop Burp, ZAP, Postman, DevTools, scripts, browser retries, two tabs or slow-network duplicate submissions. The backend must preserve the invariant.
