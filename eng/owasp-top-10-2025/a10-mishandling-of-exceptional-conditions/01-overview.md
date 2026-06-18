# A10 Overview

## Definition

Mishandling of Exceptional Conditions happens when an application does not safely prevent, detect, handle, recover from or report abnormal conditions.

The vulnerability is not the mere presence of an error. The security question is how the application behaves when the error or abnormal condition occurs.

## Examples of exceptional conditions

- missing, malformed or unexpected input reaches unsafe code,
- unexpected `null` or `undefined` values,
- database, file storage or external API failure,
- authorization service unavailable,
- session store failure,
- timeout or client disconnect,
- failed cleanup,
- partial transaction,
- unsafe retry,
- duplicate request,
- race condition,
- resource exhaustion,
- sensitive error details exposed to users.

## Expected validation failure versus exceptional condition

### Expected validation failure

A controlled rejection of invalid input that the application anticipates.

Examples:

- missing required field,
- empty value,
- `null`,
- wrong data type,
- array where a string is expected,
- unknown enum,
- malformed identifier,
- unsupported file type.

These should normally return controlled `4xx` responses such as `400 Bad Request`.

### Exceptional condition

An abnormal state that interrupts processing or leaves the application unsure what happened next.

Examples:

- database timeout,
- file-storage failure,
- authorization dependency unavailable,
- unexpected repository exception,
- transaction failure,
- client timeout after the server may already have processed the request,
- two concurrent requests changing the same state.

These may produce `5xx` responses, but the response alone does not prove safe recovery.

## A10 is not every exception

A `500` response may be evidence of an unhandled condition, but it is not automatically a vulnerability.

Review the behaviour:

- Did the response expose stack traces, SQL queries, file paths, hostnames, secrets or config?
- Did the application continue after a failed security check?
- Did it grant access when authorization could not be verified?
- Did it leave partial state?
- Could retry duplicate a side effect?
- Were resources released?
- Were repeated failures logged and monitored?

## A09 versus A10

A09 is about visibility, detection, alerting and response.

A10 is about unsafe behaviour during abnormal conditions.

```text
A10 prevention:
authorization service fails -> sensitive action is denied -> state remains safe

A09 detection:
the failure is logged and alerted when repeated or suspicious
```

Logging an unsafe failure does not make the failure safe.

## A01 / A06 versus A10

```text
Missing authorization check:
user can update another user's record -> A01 / A06

Authorization check exists but dependency times out:
backend cannot verify permission but still allows update -> A10 fail-open
```
