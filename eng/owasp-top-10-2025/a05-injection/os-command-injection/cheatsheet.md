# OS Command Injection Cheat Sheet

Use only in TryHackMe, PortSwigger Web Security Academy, local labs, or explicitly authorised testing.

This is a reasoning cheat sheet, not a raw payload dump.

## 1. Identify the feature

Examples:

- stock checker,
- feedback or email form,
- ping or diagnostic utility,
- file converter,
- image processor,
- archive function,
- PDF generator,
- backup or deployment tool.

Ask:

```text
Why might this feature start an external process?
```

## 2. Identify controlled input

Check:

- query parameters,
- form values,
- JSON fields,
- route values,
- filenames,
- cookies,
- headers,
- CMS-controlled configuration.

Do not choose a parameter only because it has weak validation. Choose it because its data flow may reach a process-execution sink.

## 3. Infer command context

Possible shapes:

```text
command USER_INPUT fixed-suffix
command "USER_INPUT" fixed-suffix
shell -c "command USER_INPUT"
fixed executable + separate argument list
```

Record which parts are facts and which are assumptions.

## 4. Establish a baseline

Before changing input:

- send the request several times,
- record normal status, body, length, and timing,
- identify expected business behaviour,
- change one value at a time.

## 5. Evidence types

### Direct output

A recognisable benign marker or username appears in the same response.

### Timing

A controlled delay is:

- repeatable,
- significantly above baseline,
- proportional to the requested value.

### File side effect

A benign command writes to an authorised lab path and a separate endpoint retrieves the file.

### Out-of-band

A unique DNS or HTTP interaction reaches an authorised OAST service.

### Weak clues

These do not prove execution by themselves:

- a single slow response,
- generic `500`,
- validation error,
- changed content length,
- failed command syntax.

## 6. Common shell operators

| Operator | Purpose or behaviour | Context note |
|---|---|---|
| `;` | command separator | common in Unix-like shells |
| `&&` | run next after success | Windows and Unix command contexts |
| `||` | run next after failure | Windows and Unix command contexts |
| `&` | separator/background behaviour | depends on shell and placement |
| `|` | pipe stdout to next command | affects data flow between commands |
| `>` | overwrite file with stdout | requires write permission |
| `>>` | append stdout to file | requires write permission |
| newline | command separator | common Unix-like context |
| `` `cmd` `` | inline command substitution | Unix-like shells |
| `$(cmd)` | inline command substitution | Unix-like shells |

PowerShell has additional parsing rules and should not be treated as identical to `cmd.exe` or Bash.

## 7. Separator placement

A closing separator can isolate the injected command:

```text
original command ; injected command ; original suffix
```

Without it, suffix text may become arguments or break syntax.

## 8. Transport encoding

For `application/x-www-form-urlencoded`:

| Character | Encoded form |
|---|---|
| space | `+` or `%20` |
| `;` | `%3B` |
| `&` inside a value | `%26` |
| `>` | `%3E` |
| `/` | `%2F` when encoding is required |

Do not encode the entire form body because `&` and `=` separate parameters.

## 9. Linux and Windows observations

Common benign information commands:

| Goal | Linux/Unix | Windows |
|---|---|---|
| current user | `whoami` | `whoami` |
| list directory | `ls` | `dir` |
| deliberate delay | `sleep`, `ping` | `timeout`, `ping` |

Payload failure does not prove the operating system. The command may be unavailable, filtered, asynchronous, quoted, or running without a shell.

## 10. Direct versus blind workflow

### Direct

```text
input change
-> command output in same response
-> verify marker is not normal content
```

### Blind timing

```text
baseline
-> controlled delay
-> repeat
-> vary delay
-> confirm proportional change
```

### Blind file

```text
confirm execution
-> identify writable authorised path
-> redirect benign stdout
-> retrieve file through known endpoint
```

### Blind OAST

```text
unique collaborator domain
-> trigger one authorised interaction
-> correlate timestamp and token
```

## 11. Stop payload guessing

Stop when tests become random.

Ask:

- What command behaviour am I trying to prove?
- What command context do I suspect?
- What result do I expect?
- What would disprove my hypothesis?
- Which single variable am I changing?

## 12. Code review sinks

Search for:

- PHP: `exec`, `system`, `shell_exec`, `passthru`, `proc_open`
- Python: `subprocess`, `os.system`, `shell=True`
- Node.js: `child_process.exec`, `spawn` with `shell: true`
- Java: `ProcessBuilder`, `Runtime.exec`, explicit `sh -c` or `cmd /c`
- .NET: `Process.Start`, `ProcessStartInfo`
- Ruby: backticks, `%x`, `system`, `Open3`
- Go: `os/exec`, especially `sh -c`

A sink is not automatically vulnerable. Trace its arguments back to request-controlled sources.

## 13. Remediation order

```text
avoid OS command
-> avoid shell
-> fixed executable
-> separate arguments
-> strict allowlist
-> prevent option injection
-> least privilege
-> timeouts and resource limits
-> logging and regression tests
```

## 14. Secondary reference policy

The PayloadBox repository may be used as a secondary lab encyclopaedia for:

- separators,
- encoding and obfuscation concepts,
- Linux and Windows variants,
- context-breaking ideas,
- blind and OOB categories.

Do not copy it as a raw payload dump. Verify techniques against PortSwigger, OWASP, official language documentation, and completed lab evidence.

Do not use destructive commands or routine reverse-shell payloads in these notes.

## References

- PortSwigger OS Command Injection: https://portswigger.net/web-security/os-command-injection
- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- PayloadBox Command Injection Payload List: https://github.com/payload-box/command-injection-payload-list
