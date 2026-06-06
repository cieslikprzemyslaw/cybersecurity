# OS Command Injection Regression Tests

These tests verify both the immediate fix and the architecture around process execution.

## Definition of done

A fix is complete when:

- user input cannot become shell syntax,
- the shell is not used unless formally justified,
- the executable is fixed,
- arguments are separate and validated,
- option injection is prevented,
- execution is restricted by permissions and limits,
- timing, file, and OAST checks no longer show command execution,
- automated tests protect the implementation from regression.

## Unit tests

### Validation and mapping

- Allowed identifiers map to approved server-side values.
- Unknown identifiers are rejected.
- Numbers enforce type and range.
- Filenames enforce expected basename and extension rules.
- Whitespace and metacharacters are rejected where not required.
- Values beginning with `-` are rejected when they could become options.
- Extremely long input is rejected before process execution.

### Command construction

- The executable is a hardcoded constant.
- Required flags are hardcoded.
- User input is not included in the executable name.
- User input is not interpolated into one command string.
- The argument list preserves one value as one argument.
- The shell option is disabled.
- `--` is used before operands where supported.

## Integration tests

### Baseline behaviour

- Valid input produces the expected business result.
- Invalid input returns a controlled `4xx` response.
- Process failures return a stable application error without raw command details.

### Shell syntax tests

Inputs containing the following must not start extra commands:

```text
;
&
&&
||
|
newline
$(...)
`...`
>
>>
```

The exact test values should be safe markers and should remain within authorised automated test environments.

### Timing tests

- Record a normal response-time range.
- Submit a benign delay-shaped marker.
- Repeat the test.
- Increase the requested delay value.
- Assert that response time does not scale with the marker.

Timing tests should allow normal CI variance and should not rely on one request.

### File side-effect tests

- Input cannot create a new file in the web root.
- Input cannot overwrite or append to an existing file.
- Temporary output is written only to a dedicated non-web-readable directory.
- The application account cannot write outside approved directories.

### Network side-effect tests

- Input cannot trigger outbound DNS or HTTP requests.
- The worker or application has only required egress access.
- Test environments can use a controlled local listener or OAST service where policy permits.

## Code-review assertions

### Node.js

- No user-controlled value reaches `child_process.exec()`.
- `spawn()` and `execFile()` do not enable `shell: true`.
- Arguments are separate array entries.
- Timeout and output limits are configured.

### Python

- No user-controlled value reaches `os.system()`.
- `subprocess` uses an argument list.
- `shell=False` is explicit or preserved.
- `check`, timeout, stdout, and stderr handling are defined.

### PHP

- User input is not concatenated into `exec()`, `system()`, `shell_exec()`, or `passthru()`.
- A native library or API replaces shell execution where possible.
- Escaping is not treated as the only security control.

### Java

- `ProcessBuilder` uses separate list elements.
- User input is not passed through `sh -c`, `bash -c`, or `cmd /c`.
- Command and flags are server-controlled.

### .NET

- `UseShellExecute` is false for process execution involving dynamic arguments.
- `ArgumentList` is preferred over building an argument string.
- The executable path is fixed and trusted.

### Ruby

- Interpolated backticks and `%x` are not used with user input.
- `Open3` or `system` receives separate program and argument values.

### Go

- User input is not passed to `sh -c`.
- `exec.CommandContext` uses a fixed executable and separate arguments.
- Context timeout and output handling are defined.

## Privilege tests

- Confirm the application runs as a dedicated non-root or non-administrator account.
- Confirm the account cannot read unrelated secrets.
- Confirm the account cannot write to the web root.
- Confirm unnecessary binaries are unavailable or blocked.
- Confirm environment variables exposed to child processes are minimised.
- Confirm network egress matches the feature's requirements.

## Observability tests

- Denied execution attempts create an appropriate security log.
- Logs include request correlation but exclude secrets and full malicious payloads where unnecessary.
- Unexpected child-process creation generates an alert in higher-risk environments.
- Repeated timing probes or process failures are detectable.
- File creation in sensitive directories is monitored.

## Example automated acceptance criteria

```text
Given a feedback email value containing shell metacharacters
When the feedback endpoint processes the request
Then no shell process is started
And no additional child command is executed
And response time remains within the normal tolerance
And no file is created in a web-readable directory
And no outbound interaction occurs
And the request receives a controlled validation response or is processed as literal data
```

## Manual verification after deployment

1. Confirm the deployed code path matches the reviewed implementation.
2. Re-run the original safe proof in an authorised environment.
3. Compare timings with baseline.
4. Check the previously writable path.
5. Check process and network telemetry.
6. Verify the application account and permissions.
7. Record evidence that the vulnerability no longer reproduces.
