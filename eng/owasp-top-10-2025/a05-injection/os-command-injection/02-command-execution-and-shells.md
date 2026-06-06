# Command Execution and Shells

## Why command-execution APIs exist

Applications sometimes need to start external programs. Programming languages therefore provide APIs for process execution.

The APIs are not automatically vulnerable. Risk depends on:

- whether a shell interprets the input,
- whether the executable is fixed,
- whether a complete command is built as one string,
- whether arguments are passed separately,
- whether the user can introduce additional flags,
- what privileges and filesystem access the process has.

## Shell command string versus argument list

### Dangerous pattern

```text
fixed command text + untrusted input + fixed suffix
-> one command string
-> shell interpretation
```

Example in Node.js:

```js
exec(`grep ${title} /var/www/html/songtitle.txt`);
```

The shell can interpret separators, redirection, substitutions, quoting, and other metacharacters inside `title`.

### Safer pattern

```text
fixed executable
+ separate argument array
+ shell disabled
```

Example in Node.js:

```js
execFile(
  "grep",
  ["--", title, "/var/www/html/songtitle.txt"],
  { timeout: 3000 },
  callback
);
```

This is safer because the program and arguments are not merged into a shell command string. It still requires validation because `grep` receives attacker-controlled data and because not every command supports the same option rules.

## THM PHP example

```php
$title = $_GET["title"];
$command = "grep $title /var/www/html/songtitle.txt";
$search = exec($command);
```

Data flow:

```text
GET title
-> $title
-> concatenated into $command
-> exec($command)
-> shell/process execution
```

Normal input:

```text
Yesterday
```

Conceptual command:

```text
grep Yesterday /var/www/html/songtitle.txt
```

A separator can split the intended command into multiple commands because the controlled value becomes shell syntax.

Code-review observations:

- `$songs = "/var/www/html/songs"` is defined in the full example but not used in the shown command.
- The file is searched through an OS utility even though structured application data would usually be stored and queried using a safer application-level mechanism.
- The exact arguments received by commands depend on where fixed suffix text appears. Parser behaviour should not be oversimplified.

## THM Python/Flask example

```python
import subprocess
from flask import Flask

app = Flask(__name__)

def execute_command(shell):
    return subprocess.Popen(
        shell,
        shell=True,
        stdout=subprocess.PIPE
    ).stdout.read()

@app.route('/<shell>')
def command_server(shell):
    return execute_command(shell)
```

Data flow:

```text
route value
-> shell parameter
-> execute_command
-> subprocess.Popen(..., shell=True)
-> shell executes the complete user-controlled string
```

A request to `/whoami` causes the server to execute `whoami` and return its output.

This is intentionally extreme. The dangerous combination is complete user control plus `shell=True`.

## Cross-language process-execution review

| Language | Security-sensitive APIs | Main review question |
|---|---|---|
| PHP | `exec()`, `system()`, `shell_exec()`, `passthru()`, `proc_open()` | Is untrusted input included in a command string? Can a non-shell API or library replace it? |
| Python | `subprocess.run()`, `Popen()`, `os.system()` | Is `shell=True` used? Are executable and arguments passed as a list? |
| Node.js | `child_process.exec()`, `execFile()`, `spawn()` | Is a command string passed to `exec`? Is `shell: true` enabled? Are arguments separate? |
| Java | `Runtime.exec()`, `ProcessBuilder` | Is a shell explicitly invoked? Are command and arguments separate array entries? |
| .NET | `Process.Start()`, `ProcessStartInfo` | Is `UseShellExecute` disabled? Is `ArgumentList` used instead of constructing one argument string? |
| Ruby | backticks, `system`, `%x`, `Open3` | Is one interpolated string used, or are program and arguments separate? |
| Go | `os/exec.Command` | Is `sh -c` used? Is the executable fixed and are arguments separate? |

## Safer examples

### Python

Unsafe:

```python
subprocess.run(f"grep {title} /var/www/html/songtitle.txt", shell=True)
```

Safer:

```python
subprocess.run(
    ["grep", "--", title, "/var/www/html/songtitle.txt"],
    shell=False,
    check=True,
    timeout=3,
)
```

### Node.js

Unsafe:

```js
exec(`grep ${title} /var/www/html/songtitle.txt`);
```

Safer:

```js
execFile(
  "grep",
  ["--", title, "/var/www/html/songtitle.txt"],
  { timeout: 3000, maxBuffer: 64 * 1024 },
  callback
);
```

### Java

Unsafe:

```java
new ProcessBuilder("sh", "-c", "grep " + title + " /var/www/html/songtitle.txt").start();
```

Safer:

```java
new ProcessBuilder(
    "grep",
    "--",
    title,
    "/var/www/html/songtitle.txt"
).start();
```

### .NET

Safer modern pattern:

```csharp
var startInfo = new ProcessStartInfo
{
    FileName = "grep",
    UseShellExecute = false,
    RedirectStandardOutput = true
};

startInfo.ArgumentList.Add("--");
startInfo.ArgumentList.Add(title);
startInfo.ArgumentList.Add("/var/www/html/songtitle.txt");
```

### Ruby

Unsafe:

```ruby
`grep #{title} /var/www/html/songtitle.txt`
```

Safer:

```ruby
stdout, stderr, status = Open3.capture3(
  "grep", "--", title, "/var/www/html/songtitle.txt"
)
```

### Go

Unsafe:

```go
exec.Command("sh", "-c", "grep "+title+" /var/www/html/songtitle.txt")
```

Safer:

```go
ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
defer cancel()

cmd := exec.CommandContext(
    ctx,
    "grep",
    "--",
    title,
    "/var/www/html/songtitle.txt",
)
```

## Argument injection

Removing the shell is a major improvement, but it is not the end of the review.

Example risk:

```text
program receives attacker-controlled argument beginning with -
-> program interprets it as an option
-> behaviour changes
```

Defences include:

- hardcode the executable,
- hardcode required flags,
- use strict allowlists,
- use `--` to end option parsing where supported,
- limit length and character set,
- do not expose raw command options to the client.

## How to reason during code review

For each process call, document:

1. source of every value,
2. whether a shell is invoked,
3. executable selection,
4. argument boundaries,
5. option parsing,
6. environment variables,
7. working directory,
8. stdin/stdout/stderr handling,
9. timeout and resource limits,
10. process privileges.

## References

- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- Python subprocess documentation: https://docs.python.org/3/library/subprocess.html
- Node.js child_process documentation: https://nodejs.org/api/child_process.html
- PHP program execution documentation: https://www.php.net/manual/en/book.exec.php
- Java ProcessBuilder documentation: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ProcessBuilder.html
- .NET ProcessStartInfo documentation: https://learn.microsoft.com/dotnet/api/system.diagnostics.processstartinfo
- Ruby Open3 documentation: https://docs.ruby-lang.org/en/master/Open3.html
- Go os/exec documentation: https://pkg.go.dev/os/exec
