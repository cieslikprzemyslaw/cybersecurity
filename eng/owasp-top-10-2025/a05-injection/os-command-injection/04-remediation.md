# OS Command Injection Remediation

## Remediation priority

The preferred order is:

1. avoid OS command execution,
2. avoid the shell,
3. use a fixed executable and separate arguments,
4. validate and allowlist values,
5. prevent argument injection,
6. reduce privileges and accessible resources,
7. apply execution limits,
8. log, monitor, and regression-test the fix.

## 1. Avoid calling OS commands

The strongest fix is to replace the command with:

- a standard-library function,
- a well-maintained application library,
- an SDK,
- a service API,
- a queue or background worker with a narrow interface.

Examples:

- use a filesystem API instead of `system("mkdir ...")`,
- use an email library or provider API instead of constructing a `mail` shell command,
- use an image-processing library binding instead of concatenating a CLI command,
- query structured data rather than shelling out to `grep`.

This removes the shell interpretation boundary rather than trying to defend it with increasingly complex string cleaning.

## 2. Avoid the shell

If an external executable is required:

- do not use `shell=True`,
- do not use `sh -c`, `bash -c`, `cmd /c`, or a PowerShell command string,
- do not use APIs that require one interpolated command string,
- use a process API that accepts executable and argument list separately.

Target design:

```text
fixed executable
+ structured arguments
+ shell disabled
```

## 3. Keep executable and arguments separate

Unsafe:

```js
exec(`convert ${source} ${destination}`);
```

Safer shape:

```js
execFile("convert", [source, destination], options, callback);
```

This prevents shell separators from being interpreted by a shell. It does not automatically make every argument safe.

## 4. Use strict allowlists

When the feature supports a limited set of values, map client choices to server-side constants.

Example:

```text
client sends: formatId=webp
server maps:  webp -> approved internal format value
```

Do not accept:

- arbitrary executable names,
- arbitrary flags,
- command fragments,
- free-form pipeline definitions,
- user-controlled working directories.

Validation should define:

- accepted type,
- exact allowed values or character set,
- minimum and maximum length,
- numeric bounds,
- filename rules,
- whether whitespace is permitted.

## 5. Prevent argument injection

A process can be unsafe even without a shell if attacker input becomes an option.

Example:

```text
expected operand: filename
attacker value:   --unexpected-option
```

Controls:

- use `--` to terminate option parsing where supported,
- hardcode required flags,
- reject option-like values when not valid,
- validate according to the target program's argument grammar,
- avoid exposing raw CLI features through an HTTP API.

## 6. Apply least privilege

The application process should:

- run as a dedicated non-administrative account,
- read only required files,
- write only to required directories,
- avoid write access to web roots,
- have minimal network access,
- execute only necessary binaries,
- receive a minimal environment.

Least privilege reduces impact. It does not repair unsafe command construction.

## 7. Add timeouts and resource limits

Process execution should define:

- execution timeout,
- maximum stdout and stderr size,
- memory and CPU constraints where available,
- concurrency limits,
- controlled working directory,
- cancellation behaviour,
- safe cleanup of temporary files.

These controls reduce denial-of-service and runaway-process risk.

## 8. Handle output and errors safely

- Do not return raw process errors or command strings to users.
- Do not include secrets in logs.
- Treat stdout and stderr as untrusted data.
- Do not render command output as HTML without appropriate output encoding.
- Use controlled application errors.
- Record enough internal context for investigation without exposing it externally.

## 9. Logging and monitoring

Monitor for:

- repeated process failures,
- shell metacharacters in fields that should not contain them,
- unusual execution duration,
- unexpected child processes,
- writes to unusual directories,
- outbound DNS or HTTP activity,
- spikes in process creation,
- denied execution attempts.

Logging is detection and response support, not the primary prevention control.

## 10. Regression testing

A complete fix should prove:

- the shell is no longer invoked,
- executable selection is server-controlled,
- arguments are separate and validated,
- separators do not create extra commands,
- option-like input does not alter program behaviour,
- timing tests do not produce controlled delays,
- unexpected files are not created,
- outbound interactions do not occur,
- process privileges and filesystem access are restricted.

## Why sanitisation alone is weak

Shell parsing varies across:

- operating systems,
- shells,
- quoting contexts,
- encodings,
- command substitutions,
- environment expansion,
- application decode layers.

A blacklist of dangerous characters is easy to make incomplete. Escaping can also stop shell injection while leaving argument injection possible.

Therefore:

```text
avoid shell > structured process API > allowlist > escaping as last-resort support
```

## Developer review example

Weak remediation:

> We removed semicolons from the email value.

Stronger remediation:

> The feedback feature no longer constructs a shell command. It uses the provider's email SDK. The destination address is configured server-side, the user email is validated as an email value, the application runs as a restricted account, and regression tests verify that metacharacters cannot cause timing, file, or network side effects.

## References

- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- PortSwigger prevention guidance: https://portswigger.net/web-security/os-command-injection
