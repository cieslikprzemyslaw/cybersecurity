# Error Handling and Information Disclosure

## External response

A safe external error response should contain enough information for the client to handle the error, but not enough to reveal internals.

Useful fields:

- suitable HTTP status,
- generic message,
- stable application error code where useful,
- request ID or correlation ID.

Do not expose:

- stack traces,
- SQL queries,
- filesystem paths,
- secrets or tokens,
- internal hostnames,
- sensitive configuration,
- raw exception objects,
- full request or response bodies.

## Generic response does not prove recovery

```text
500 Something went wrong
```

can be safe from an information-disclosure perspective, but it does not prove:

- rollback happened,
- files were cleaned up,
- a payment did not go through,
- an email was not sent,
- a role was not changed,
- resources were released,
- retry is safe.

The response is what the client saw. It is not proof of backend state.

## Internal logs

Internal logs should contain safe context for investigation:

- request/correlation ID,
- operation name,
- actor ID where available,
- target resource ID,
- safe reason code,
- result,
- timestamp,
- environment.

Logs must still avoid secrets and unnecessary personal data.

## Global error handler

A global error handler is useful as a fallback for unexpected exceptions. It can prevent stack trace disclosure.

It does not replace local handling at the layer that understands the operation. It does not automatically roll back multi-step workflows, delete partial files or reverse third-party side effects.

## Try/catch is not enough

Unsafe patterns:

```text
catch error -> log -> continue as success
catch error -> ignore -> return 200
catch error -> retry unsafe action repeatedly
catch error -> leave partial state
```

Safer handling:

```text
catch error
    -> stop sensitive operation
    -> rollback or clean up where possible
    -> avoid continuing from unknown state
    -> return controlled response
    -> log safe internal evidence
```
