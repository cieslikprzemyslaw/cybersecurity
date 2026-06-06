# Command Execution i shelle

## Dlaczego API uruchamiania procesów istnieją

Aplikacje czasem muszą uruchamiać zewnętrzne programy, więc języki programowania dostarczają API do process execution.

Same API nie są automatycznie podatne. Ryzyko zależy od tego:

- czy shell interpretuje input,
- czy executable jest stały,
- czy kompletna komenda jest budowana jako jeden string,
- czy argumenty są przekazywane osobno,
- czy użytkownik może wprowadzić dodatkowe flagi,
- jakie uprawnienia i dostęp do filesystemu ma proces.

## String komendy vs lista argumentów

### Niebezpieczny wzorzec

```text
stały tekst komendy + niezaufany input + stały suffix
-> jeden string komendy
-> interpretacja przez shell
```

Przykład Node.js:

```js
exec(`grep ${title} /var/www/html/songtitle.txt`);
```

Shell może interpretować separatory, redirection, substitutions, quoting i inne metaznaki w `title`.

### Bezpieczniejszy wzorzec

```text
stały executable
+ osobna tablica argumentów
+ shell disabled
```

Przykład Node.js:

```js
execFile(
  "grep",
  ["--", title, "/var/www/html/songtitle.txt"],
  { timeout: 3000 },
  callback
);
```

To jest bezpieczniejsze, bo program i argumenty nie są łączone w jeden string shella. Nadal wymaga walidacji, bo `grep` dostaje dane kontrolowane przez użytkownika i nie każdy program ma takie same reguły opcji.

## Przykład THM PHP

```php
$title = $_GET["title"];
$command = "grep $title /var/www/html/songtitle.txt";
$search = exec($command);
```

Data flow:

```text
GET title
-> $title
-> doklejone do $command
-> exec($command)
-> shell/process execution
```

Normalny input:

```text
Yesterday
```

Koncepcyjna komenda:

```text
grep Yesterday /var/www/html/songtitle.txt
```

Separator może podzielić zamierzoną komendę na kilka komend, bo kontrolowana wartość staje się składnią shella.

Obserwacje do code review:

- `$songs = "/var/www/html/songs"` jest zdefiniowane w pełnym przykładzie, ale nie jest użyte w pokazanej komendzie.
- Plik jest przeszukiwany przez narzędzie OS, chociaż strukturalne dane aplikacji zwykle powinny być przechowywane i odpytywane bezpieczniejszym mechanizmem aplikacyjnym.
- Dokładne argumenty komend zależą od położenia stałego suffixu. Nie warto nadmiernie upraszczać zachowania parsera.

## Przykład THM Python/Flask

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
-> parametr shell
-> execute_command
-> subprocess.Popen(..., shell=True)
-> shell wykonuje pełny string kontrolowany przez użytkownika
```

Request do `/whoami` powoduje wykonanie `whoami` i zwrócenie outputu.

To jest przykład celowo skrajny. Niebezpieczne połączenie to pełna kontrola użytkownika plus `shell=True`.

## Cross-language review process execution

| Język | Security-sensitive APIs | Główne pytanie review |
|---|---|---|
| PHP | `exec()`, `system()`, `shell_exec()`, `passthru()`, `proc_open()` | Czy niezaufany input trafia do stringa komendy? Czy biblioteka albo API może zastąpić shell? |
| Python | `subprocess.run()`, `Popen()`, `os.system()` | Czy użyto `shell=True`? Czy executable i argumenty są listą? |
| Node.js | `child_process.exec()`, `execFile()`, `spawn()` | Czy do `exec` trafia string komendy? Czy włączono `shell: true`? Czy argumenty są osobne? |
| Java | `Runtime.exec()`, `ProcessBuilder` | Czy shell jest wywołany jawnie? Czy komenda i argumenty są osobnymi elementami tablicy? |
| .NET | `Process.Start()`, `ProcessStartInfo` | Czy `UseShellExecute` jest wyłączone? Czy użyto `ArgumentList` zamiast budowania jednego stringa argumentów? |
| Ruby | backticks, `system`, `%x`, `Open3` | Czy użyto jednego interpolowanego stringa, czy osobnego programu i argumentów? |
| Go | `os/exec.Command` | Czy użyto `sh -c`? Czy executable jest stały, a argumenty osobne? |

## Bezpieczniejsze przykłady

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

Bezpieczniejszy nowoczesny wzorzec:

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

Usunięcie shella to duża poprawa, ale review się na tym nie kończy.

Przykładowe ryzyko:

```text
program otrzymuje argument kontrolowany przez atakującego zaczynający się od -
-> program interpretuje go jako opcję
-> zachowanie się zmienia
```

Kontrole:

- hardcode executable,
- hardcode wymagane flagi,
- używaj ścisłych allowlist,
- używaj `--` do zakończenia parsowania opcji tam, gdzie program to wspiera,
- ogranicz długość i zestaw znaków,
- nie wystawiaj surowych opcji CLI przez HTTP API.

## Jak myśleć podczas code review

Dla każdego process call zapisz:

1. źródło każdej wartości,
2. czy shell jest wywoływany,
3. wybór executable,
4. granice argumentów,
5. parsowanie opcji,
6. zmienne środowiskowe,
7. working directory,
8. obsługę stdin/stdout/stderr,
9. timeouty i limity zasobów,
10. uprawnienia procesu.

## Referencje

- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- Python subprocess documentation: https://docs.python.org/3/library/subprocess.html
- Node.js child_process documentation: https://nodejs.org/api/child_process.html
- PHP program execution documentation: https://www.php.net/manual/en/book.exec.php
- Java ProcessBuilder documentation: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ProcessBuilder.html
- .NET ProcessStartInfo documentation: https://learn.microsoft.com/dotnet/api/system.diagnostics.processstartinfo
- Ruby Open3 documentation: https://docs.ruby-lang.org/en/master/Open3.html
- Go os/exec documentation: https://pkg.go.dev/os/exec
