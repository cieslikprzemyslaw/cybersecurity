# SSTI Regression Tests

## Purpose

Regression tests should prove that untrusted input remains data and cannot become template syntax after a fix or future refactor.

## Test 1: Expression remains text

Send an engine-specific harmless expression through the affected input.

Expected secure result:

- expression is displayed literally or safely encoded, or
- the application rejects it according to business rules.

Unexpected vulnerable result:

- expression is replaced by its calculated value.

Example assertion model:

```text
input contains template expression for 7 * 7
response must not contain 49 as the evaluated replacement
```

This test distinguishes reflection from template evaluation.

## Test 2: Static template behaviour remains functional

Send a normal allowed message or status value.

Expected:

- correct message is displayed,
- no template error,
- output encoding remains correct.

Security fixes should not break legitimate rendering.

## Test 3: Server-selected message mapping

When the application uses a status identifier:

```text
status=out_of_stock
```

Expected:

- server returns the fixed out-of-stock message.

For an unknown value:

```text
status=<template syntax or unknown key>
```

Expected:

- safe default or validation error,
- no dynamic compilation,
- no expression evaluation.

## Test 4: No side effects

Supply input containing template-like syntax that would call a harmless test helper in a deliberately instrumented test environment.

Expected:

- helper is not called,
- no file is created, modified, or deleted,
- no outbound request occurs,
- no child process starts.

Do not use destructive production tests.

## Test 5: Error handling

Supply malformed template-like input.

Expected:

- no stack trace,
- no template filename,
- no engine internals,
- no filesystem path disclosure,
- protected server logs contain enough detail for debugging.

## Test 6: Stored input

When the value can be saved in a CMS, profile, database, or email template field:

1. Store template-looking text.
2. Render every feature that later consumes it.
3. Confirm the value stays data in every rendering path.

This detects second-order SSTI where input is safe at submission but evaluated later.

## Code-level tests

- Assert the application loads templates from approved static resources.
- Reject or flag dynamic template compilation involving request values.
- Verify only approved variables/helpers exist in template context.
- Verify sandbox policy when user-authored templates are an intentional feature.
- Confirm the application process has least-privilege filesystem and network permissions.

## Example security test wording

> Given a user-controlled `message` value containing ERB syntax, when the page is rendered, the response must contain the value as text and must not contain the evaluated result or produce any filesystem side effect.
