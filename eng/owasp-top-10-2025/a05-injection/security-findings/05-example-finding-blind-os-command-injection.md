# Blind OS Command Injection in Feedback Submission

## Summary

The feedback submission endpoint passed a user-controlled email value into a shell command. Shell metacharacters in the `email` parameter changed operating-system command execution.

The application did not return command output directly, but execution was confirmed through repeatable timing behaviour and output redirection to a web-readable file.

## Suggested severity

**High**

The exact severity depends on the privileges of the application process, accessible files, network access, available binaries, and surrounding infrastructure. Arbitrary command execution as even a low-privilege web user can expose application data and create a path to broader compromise.

## Affected component

```text
POST /feedback/submit
Parameter: email
```

## Vulnerable behaviour

The endpoint accepted an email value and used it as part of a shell command. A command separator was interpreted as shell syntax rather than ordinary email data.

Conceptual vulnerable flow:

```text
user-controlled email
-> backend constructs command string
-> shell interprets command separators
-> additional OS command executes
```

The exact original backend command was not visible. It was likely related to processing or sending feedback, but that remains an inference rather than a confirmed fact.

## Evidence

### Timing evidence

A normal request completed in milliseconds.

A controlled test that inserted a loopback `ping` produced approximately:

```text
ping count 5  -> about 4.1 seconds
ping count 10 -> about 9.3 seconds
```

The same endpoint returned `500 Internal Server Error` with `"Could not save"`, but the delay scaled with the controlled packet count. This repeatable and proportional timing behaviour provided strong evidence that the injected command executed.

### File-based evidence

The output of `whoami` was redirected to a writable image directory:

```text
/var/www/images/output.txt
```

The file was then retrieved through:

```http
GET /image?filename=output.txt
```

The response returned:

```text
peter-VqYsRp
```

The random suffix was specific to the lab instance. The result demonstrated that the injected command executed as the application process user.

## Why the vulnerability was blind

The response from `POST /feedback/submit` did not contain command output. Evidence was recovered through side effects:

- response timing,
- file creation,
- a separate file retrieval request.

The `/image` endpoint was not the command-execution sink. It only exposed the file created by the injected command.

## Root cause

Untrusted input was allowed to influence a shell command string. The application failed to preserve a security boundary between data and executable shell syntax.

The root cause was not merely "missing sanitisation". The unsafe design was using user-controlled input in shell-interpreted command construction.

## Impact

Depending on process privileges and environment controls, an attacker could potentially:

- read application-accessible files,
- modify writable files,
- access secrets stored in configuration or environment data,
- invoke installed utilities,
- make outbound network requests,
- disrupt application availability,
- use the compromised process as a starting point for further attacks.

No destructive commands were required to prove the issue in the authorised lab.

## Remediation

1. Remove the OS command call if a language library, email SDK, queue, or service API can perform the task.
2. If process execution is unavoidable, use a fixed executable and pass each argument separately.
3. Do not invoke a shell and do not build one command string through concatenation.
4. Hardcode required flags and map limited user choices to server-side allowlisted values.
5. Prevent argument injection, including option-like values beginning with `-`.
6. Run the application as a dedicated low-privilege account.
7. Restrict filesystem write access, especially to web-readable directories.
8. Apply execution timeouts, output limits, and resource controls.
9. Log and alert on unexpected child processes, repeated execution failures, suspicious delays, and unusual file creation.

## Regression tests

- Shell separators in `email` must be rejected or treated as literal data.
- A test marker must not produce measurable execution delay.
- Increasing a supplied delay value must not increase response time proportionally.
- The input must not create files in `/var/www/images/` or another application directory.
- The input must not trigger outbound DNS or HTTP interactions.
- Code review must confirm that the feature does not use a shell command string.
- Process execution, if still required, must use a fixed executable and separate arguments with the shell disabled.
- The application account must not have unnecessary read, write, or execute permissions.

## Authorised reproduction summary

1. Capture a normal feedback submission.
2. Establish a baseline response time.
3. Modify only the suspected parameter with a benign timing test.
4. Repeat the test and vary the controlled delay to confirm proportional behaviour.
5. Redirect a benign command result to the documented writable lab directory.
6. Retrieve the file through the lab's image endpoint.
7. Record the response, timing, output, assumptions, and exact root cause.

## References

- PortSwigger OS Command Injection: https://portswigger.net/web-security/os-command-injection
- PortSwigger blind output redirection lab: https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection
- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
