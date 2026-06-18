# Fail-open and Fail-closed

## Fail-open

Fail-open means the application allows a sensitive operation when a security control cannot make a safe decision.

Example:

```text
authorization service timeout
    -> backend cannot verify permission
    -> backend allows the request anyway
    -> protected state is changed
```

The application treats uncertainty as permission.

## Fail-closed

Fail-closed means the application denies or stops the sensitive operation when it cannot confirm the security rule.

Example:

```text
authorization service timeout
    -> backend cannot verify permission
    -> backend denies the operation
    -> protected state is not changed
    -> internal evidence is logged
```

Fail-closed does not mean crashing the whole application. It means preserving the security rule and returning to a known safe state.

## Review questions

1. What security rule must remain true?
2. Which dependency or check could fail?
3. What happens when the application cannot verify the rule?
4. Does the code allow the action anyway?
5. Does the system return to a known safe state?

## Common fail-open patterns

- authorization service fails and access is allowed,
- session lookup fails and user is treated as authenticated,
- file scanner fails and upload is accepted as safe,
- permission lookup fails and admin behaviour is enabled,
- catch block logs an error but continues the operation,
- validation throws and unsafe defaults are used.

## Frontend note

The UI must not show a sensitive operation as successful without backend confirmation. After timeout, the safe UI state is “unable to confirm”, followed by a refresh of the authoritative backend state.
