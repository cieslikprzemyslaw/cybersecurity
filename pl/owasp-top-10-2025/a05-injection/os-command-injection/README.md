# OS Command Injection

OS Command Injection występuje wtedy, gdy niezaufany input trafia do komendy systemowej, shella, command interpretera albo niebezpiecznej ścieżki uruchamiania procesu i zmienia to, co system wykonuje.

```text
input kontrolowany przez użytkownika
-> kod backendu
-> budowa komendy albo argumentów procesu
-> shell albo process execution
-> zmienione zachowanie systemu operacyjnego
```

## Ukończona nauka

- teoria OS Command Injection z TryHackMe,
- przykład PHP z przepływem danych do `exec()`,
- przykład Python Flask z `subprocess.Popen(..., shell=True)`,
- direct/verbose i blind command injection,
- timing, file redirection i koncepcje OAST,
- lab PortSwigger simple command injection,
- lab PortSwigger blind output redirection.

## Zacznij tutaj

1. [Overview](01-overview.md)
2. [Command execution i shelle](02-command-execution-and-shells.md)
3. [Direct i blind command injection](03-direct-and-blind-command-injection.md)
4. [Remediacja](04-remediation.md)
5. [Cheat sheet](cheatsheet.md)
6. [Testy regresji](regression-tests.md)

## Laby

- [Simple OS command injection](labs/01-simple-command-injection.md)
- [Blind OS command injection z output redirection](labs/02-blind-command-injection-output-redirection.md)
- [Porównanie labów](labs/summary.md)

## Główne pytania

Dla każdego podejrzenia ustal:

- Jaka funkcja jest testowana?
- Jaki input kontroluję?
- Dokąd trafia?
- Jakie process API albo shell może go otrzymać?
- Czy budowany jest pełny string komendy?
- Czy shell jest używany, czy argumenty są przekazywane osobno?
- Co jest faktem, a co pozostaje założeniem?
- Jaki dowód potwierdza wykonanie?
- Czy output jest direct, blind, time-based, file-based albo out-of-band?
- Jaka jest przyczyna źródłowa?
- Jak funkcja powinna być zaimplementowana bezpiecznie?
- Jakie testy regresji powinny zapobiec powrotowi problemu?

## Kluczowe rozróżnienie

OS Command Injection to nie tylko „zła walidacja”.

Główna awaria polega na tym, że niezaufane dane mogą stać się wykonywalną składnią komendy albo niebezpiecznymi argumentami procesu.

Preferowana poprawka to usunięcie shell execution, a nie coraz bardziej skomplikowana sanitizacja stringów wysyłanych do shella.

## Zakres i bezpieczeństwo

Wszystkie przykłady i obserwacje dotyczą TryHackMe, PortSwigger Web Security Academy, lokalnych labów albo jawnie autoryzowanych testów.

Notatki używają nieszkodliwych komend dowodowych i nie traktują reverse shelli ani destrukcyjnych komend jako rutynowych technik wykrywania.

## Referencje

- TryHackMe Command Injection: https://tryhackme.com/room/oscommandinjection
- PortSwigger OS Command Injection: https://portswigger.net/web-security/os-command-injection
- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- PayloadBox list, tylko jako drugorzędna referencja labowa: https://github.com/payload-box/command-injection-payload-list
