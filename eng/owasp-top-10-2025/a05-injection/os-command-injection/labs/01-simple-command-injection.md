# Lab 1 - Simple OS Command Injection

## Platform

PortSwigger Web Security Academy

Lab: https://portswigger.net/web-security/os-command-injection/lab-simple

## Scope

Authorised training lab only.

## Lab goal

Use the stock-checking feature to execute a benign command and identify the operating-system user running the application process.

## Vulnerable feature

```text
POST /product/stock
```

The request contained:

```text
productId=1&storeId=1
```

## Controlled input

Both values were client-controlled, but `storeId` was selected for the test.

## Initial hypothesis

The stock checker might invoke a legacy script or command using product and store identifiers as arguments.

Possible conceptual command:

```text
stock-report <productId> <storeId>
```

The exact backend command was not visible, so this remained a hypothesis.

## Controlled test

The `storeId` value was changed so a shell separator introduced a benign `whoami` command.

The response contained:

```text
62
peter-H8uEby
```

## Evidence

- `62` was the normal stock result.
- `peter-H8uEby` was recognisable output from `whoami`.
- Both appeared in the same HTTP response.

This was direct or verbose OS Command Injection.

## What the result proved

```text
controlled storeId
-> shell separator interpreted
-> additional command executed
-> stdout returned in HTTP response
```

The successful separator strongly indicated shell-like interpretation, but it did not identify the exact shell implementation.

## Root cause

User-controlled input was included in a shell-interpreted command without preserving separation between data and command syntax.

## Impact

Arbitrary command execution as the application process user can expose application-readable data and create a path to broader compromise depending on privileges and environment access.

## Remediation

- Replace the external command with a safe internal API where possible.
- If a process is required, use a fixed executable and pass product and store IDs as separate validated arguments.
- Do not invoke a shell.
- Enforce numeric type and range.
- Run the process with least privilege and execution limits.
- Add regression tests for separators, option-like values, timing, files, and outbound interactions.

## Learning note

This lab was intentionally simple. The key lesson was that direct command output is high-quality evidence. It should still be tied to a clear data-flow explanation rather than treated as only a successful payload.
