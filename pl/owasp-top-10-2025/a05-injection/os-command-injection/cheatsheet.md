# OS Command Injection Cheat Sheet

Używaj tylko w TryHackMe, PortSwigger Web Security Academy, lokalnych labach albo jawnie autoryzowanych testach.

To jest cheat sheet do myślenia, nie surowy payload dump.

## 1. Zidentyfikuj funkcję

Przykłady:

- stock checker,
- formularz feedbacku albo email,
- ping albo narzędzie diagnostyczne,
- konwerter plików,
- image processor,
- funkcja archiwum,
- generator PDF,
- backup albo deployment tool.

Pytanie:

```text
Dlaczego ta funkcja mogłaby uruchamiać zewnętrzny proces?
```

## 2. Zidentyfikuj kontrolowany input

Sprawdź:

- query parameters,
- wartości formularza,
- pola JSON,
- route values,
- nazwy plików,
- cookies,
- nagłówki,
- konfigurację kontrolowaną przez CMS.

Nie wybieraj parametru tylko dlatego, że ma słabą walidację. Wybierz go, jeśli jego data flow może prowadzić do process-execution sink.

## 3. Ustal kontekst komendy

Możliwe kształty:

```text
command USER_INPUT fixed-suffix
command "USER_INPUT" fixed-suffix
shell -c "command USER_INPUT"
fixed executable + separate argument list
```

Zapisuj, co jest faktem, a co założeniem.

## 4. Ustal baseline

Przed zmianą inputu:

- wyślij request kilka razy,
- zapisz normalny status, body, length i timing,
- ustal oczekiwane zachowanie biznesowe,
- zmieniaj jedną wartość naraz.

## 5. Typy dowodów

### Direct output

Rozpoznawalny nieszkodliwy marker albo username pojawia się w tej samej odpowiedzi.

### Timing

Kontrolowane opóźnienie jest:

- powtarzalne,
- wyraźnie powyżej baseline,
- proporcjonalne do żądanej wartości.

### File side effect

Nieszkodliwa komenda zapisuje wynik w autoryzowanej ścieżce labowej, a osobny endpoint pobiera plik.

### Out-of-band

Unikalna interakcja DNS albo HTTP trafia do autoryzowanego serwisu OAST.

### Słabe wskazówki

Same w sobie nie dowodzą wykonania:

- jeden wolny response,
- generyczne `500`,
- validation error,
- zmieniona długość odpowiedzi,
- nieudana składnia komendy.

## 6. Częste operatory shella

| Operator | Cel albo zachowanie | Uwaga kontekstowa |
|---|---|---|
| `;` | separator komend | typowy w shellach Unix-like |
| `&&` | uruchom kolejną po sukcesie | konteksty Windows i Unix |
| `||` | uruchom kolejną po porażce | konteksty Windows i Unix |
| `&` | separator albo background | zależy od shella i położenia |
| `|` | pipe stdout do kolejnej komendy | zmienia przepływ danych |
| `>` | nadpisz plik stdoutem | wymaga prawa zapisu |
| `>>` | dopisz stdout do pliku | wymaga prawa zapisu |
| newline | separator komend | typowy Unix-like context |
| `` `cmd` `` | inline command substitution | Unix-like shells |
| `$(cmd)` | inline command substitution | Unix-like shells |

PowerShell ma dodatkowe reguły parsowania i nie powinien być traktowany jak `cmd.exe` albo Bash.

## 7. Położenie separatora

Separator zamykający może odizolować wstrzykniętą komendę:

```text
oryginalna komenda ; wstrzyknięta komenda ; oryginalny suffix
```

Bez niego suffix może stać się argumentem albo zepsuć składnię.

## 8. Transport encoding

Dla `application/x-www-form-urlencoded`:

| Znak | Encoded form |
|---|---|
| spacja | `+` albo `%20` |
| `;` | `%3B` |
| `&` wewnątrz wartości | `%26` |
| `>` | `%3E` |
| `/` | `%2F`, gdy encoding jest wymagany |

Nie koduj całego body formularza, bo `&` i `=` rozdzielają parametry.

## 9. Linux i Windows observations

Nieszkodliwe komendy informacyjne:

| Cel | Linux/Unix | Windows |
|---|---|---|
| aktualny użytkownik | `whoami` | `whoami` |
| lista katalogu | `ls` | `dir` |
| celowe opóźnienie | `sleep`, `ping` | `timeout`, `ping` |

Nieudany payload nie dowodzi systemu operacyjnego. Komenda może być niedostępna, filtrowana, asynchroniczna, w cudzysłowie albo uruchamiana bez shella.

## 10. Direct vs blind workflow

### Direct

```text
zmiana inputu
-> output komendy w tej samej odpowiedzi
-> sprawdź, że marker nie jest normalną treścią
```

### Blind timing

```text
baseline
-> kontrolowane opóźnienie
-> powtórz
-> zmień opóźnienie
-> potwierdź proporcjonalną zmianę
```

### Blind file

```text
potwierdź wykonanie
-> wskaż zapisywalną autoryzowaną ścieżkę
-> przekieruj nieszkodliwy stdout
-> pobierz plik znanym endpointem
```

### Blind OAST

```text
unikalna domena collaborator
-> jedna autoryzowana interakcja
-> korelacja timestampu i tokenu
```

## 11. Przestań zgadywać payloady

Zatrzymaj się, gdy testy stają się losowe.

Zapytaj:

- Jakie zachowanie komendy próbuję udowodnić?
- Jaki kontekst komendy podejrzewam?
- Jakiego wyniku oczekuję?
- Co obaliłoby moją hipotezę?
- Którą jedną zmienną zmieniam?

## 12. Sinki do code review

Szukaj:

- PHP: `exec`, `system`, `shell_exec`, `passthru`, `proc_open`
- Python: `subprocess`, `os.system`, `shell=True`
- Node.js: `child_process.exec`, `spawn` z `shell: true`
- Java: `ProcessBuilder`, `Runtime.exec`, jawne `sh -c` albo `cmd /c`
- .NET: `Process.Start`, `ProcessStartInfo`
- Ruby: backticks, `%x`, `system`, `Open3`
- Go: `os/exec`, szczególnie `sh -c`

Sink nie jest automatycznie podatny. Prześledź jego argumenty do źródeł kontrolowanych przez request.

## 13. Kolejność remediacji

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

## 14. Polityka drugorzędnych referencji

Repozytorium PayloadBox może być używane jako drugorzędna encyklopedia labowa dla:

- separatorów,
- encodingu i obfuscation concepts,
- wariantów Linux i Windows,
- pomysłów na context breaking,
- kategorii blind i OOB.

Nie kopiuj go jako surowego payload dump. Techniki weryfikuj względem PortSwigger, OWASP, oficjalnej dokumentacji języków i ukończonych labów.

Nie używaj destrukcyjnych komend ani rutynowych reverse-shell payloads w tych notatkach.

## Referencje

- PortSwigger OS Command Injection: https://portswigger.net/web-security/os-command-injection
- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- PayloadBox Command Injection Payload List: https://github.com/payload-box/command-injection-payload-list
