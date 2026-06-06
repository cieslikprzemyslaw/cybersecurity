# OS Command Injection - Overview

## Definicja

OS Command Injection występuje wtedy, gdy oprogramowanie używa inputu zależnego od użytkownika do zbudowania albo kontrolowania komendy systemowej, a input zmienia zamierzone wykonanie.

Często mówi się o shell injection, ale review powinno objąć też niebezpieczne argumenty procesu, nawet gdy shell nie jest używany.

## Model mentalny

```text
kontrolowany input
-> budowa komendy albo argumentów w backendzie
-> shell albo process execution API
-> zmienione wykonanie
-> obserwowalny dowód albo impact
```

## Kontrolowany input

Możliwe źródła:

- query parameters,
- pola formularzy,
- właściwości JSON body,
- route parameters,
- nagłówki,
- cookies,
- nazwy plików,
- metadane uploadu,
- ustawienia webhooków,
- wartości CMS,
- konfiguracja scheduled jobs.

Wartość nie jest niebezpieczna tylko dlatego, że kontroluje ją użytkownik. Ryzyko pojawia się, gdy trafia do security-sensitive sink i może wpłynąć na składnię komendy, executable, flagi, argumenty, środowisko, working directory, redirection albo ścieżki plików.

## Typowe funkcje uruchamiające procesy

Command execution może pojawić się w:

- diagnostyce sieciowej, np. ping albo traceroute,
- konwersji obrazów i dokumentów,
- tworzeniu albo rozpakowywaniu archiwów,
- przetwarzaniu wideo,
- generowaniu PDF,
- wysyłce emaili,
- backupie i restore,
- wrapperach antywirusowych albo skanujących,
- narzędziach Git lub deploymentu,
- integracjach z drukarkami i urządzeniami,
- legacy scripts,
- endpointach administracyjnych i supportowych.

Produkcyjna aplikacja zwykle powinna używać biblioteki, SDK albo service API zamiast wywoływać narzędzie systemowe przez shell.

## SQL Injection vs OS Command Injection

| Obszar | SQL Injection | OS Command Injection |
|---|---|---|
| Interpreter | silnik zapytań bazy danych | shell, command interpreter albo process API |
| Zamierzona operacja | zapytanie do bazy | uruchomienie procesu systemowego |
| Zmienione zachowanie | logika query albo zwrócone rekordy | uruchomiony program, argumenty, redirection, filesystem albo network side effect |
| Typowy dowód | błąd bazy, zwrócone wiersze, boolean oracle, delay | output, delay, side effect w pliku, outbound interaction |
| Główny wzorzec bezpieczny | parameterized queries | unikaj komend OS; jeśli konieczne, stały executable i osobne argumenty bez shella |

Wspólny problem injection:

```text
dane przekraczają granicę i stają się wykonywalną instrukcją
```

## Shell vs process execution

Shell interpretuje składnię: separatory, pipe'y, substitutions, quoting, redirection, zmienne i wildcards.

Przykłady:

- `/bin/sh`,
- Bash,
- `cmd.exe`,
- PowerShell.

Process API może uruchomić executable bez użycia shella. Wtedy separatory shella mogą pozostać zwykłymi znakami argumentu. Nadal jednak niebezpieczne argumenty mogą zmienić zachowanie wybranego programu. To jest argument injection.

## Przyczyna źródłowa

Mocny opis root cause:

> Niezaufany input mógł wpłynąć na wykonywalną składnię komendy albo security-sensitive argumenty procesu, ponieważ aplikacja nie utrzymała separacji między danymi a kontrolą wykonania.

Słabe opisy:

- „Payload zawierał średnik.”
- „Użytkownik podał złe dane.”
- „Brakowało walidacji.”

To są symptomy albo brakujące kontrolki, nie opis granicy projektowej, która zawiodła.

## Impact

Impact zależy od konta aplikacji i środowiska. Możliwe skutki:

- ujawnienie plików czytelnych dla aplikacji,
- modyfikacja zapisywalnych plików,
- dostęp do konfiguracji lub sekretów,
- użycie zainstalowanych narzędzi,
- outbound network access,
- denial of service,
- przejęcie aplikacji,
- ruch w stronę innych systemów wewnętrznych, jeśli pozwalają na to zaufanie i sieć.

Least privilege zmniejsza impact, ale nie usuwa podatności.

## Model dowodów

Dowód powinien pokazywać, że kontrolowany input zmienił wykonanie systemowe.

Silne przykłady:

- rozpoznawalny output komendy w odpowiedzi HTTP,
- powtarzalne i proporcjonalne opóźnienie,
- przewidywalny plik utworzony przez output redirection,
- autoryzowany DNS albo HTTP callback,
- zamierzona akcja aplikacji zastąpiona albo uzupełniona side effectem komendy.

Słabe przykłady:

- jeden wolny request,
- generyczna odpowiedź `500`,
- błąd walidacji,
- zmieniona odpowiedź bez związku z process execution,
- nieudany payload użyty jako „dowód” systemu operacyjnego.

## Takeaway

OS Command Injection to problem data flow i granicy wykonania. Kluczowa umiejętność nie polega na pamiętaniu separatorów, ale na prześledzeniu inputu do sinka komendy/procesu, ustaleniu czy shell go interpretuje, zebraniu wiarygodnego dowodu i zaproponowaniu implementacji, która usuwa shell boundary tam, gdzie to możliwe.
