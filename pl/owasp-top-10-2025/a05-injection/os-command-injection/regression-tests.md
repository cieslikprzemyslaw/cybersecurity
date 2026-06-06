# OS Command Injection - Testy regresji

Te testy weryfikują zarówno bezpośrednią poprawkę, jak i architekturę wokół process execution.

## Definition of done

Poprawka jest kompletna, gdy:

- user input nie może stać się składnią shella,
- shell nie jest używany bez formalnego uzasadnienia,
- executable jest stały,
- argumenty są osobne i zwalidowane,
- argument injection jest zablokowane,
- wykonanie jest ograniczone uprawnieniami i limitami,
- timing, file i OAST checks nie pokazują już command execution,
- testy automatyczne chronią implementację przed regresją.

## Unit tests

### Walidacja i mapowanie

- Dozwolone identyfikatory mapują się na zatwierdzone wartości po stronie serwera.
- Nieznane identyfikatory są odrzucane.
- Liczby egzekwują typ i zakres.
- Nazwy plików egzekwują oczekiwany basename i rozszerzenie.
- Whitespace i metaznaki są odrzucane tam, gdzie nie są wymagane.
- Wartości zaczynające się od `-` są odrzucane, jeśli mogłyby stać się opcjami.
- Bardzo długi input jest odrzucany przed process execution.

### Budowa komendy

- Executable jest hardcoded constant.
- Wymagane flagi są hardcoded.
- User input nie trafia do nazwy executable.
- User input nie jest interpolowany do jednego stringa komendy.
- Lista argumentów zachowuje jedną wartość jako jeden argument.
- Opcja shella jest wyłączona.
- `--` jest używane przed operandami tam, gdzie program to wspiera.

## Integration tests

### Baseline behaviour

- Poprawny input daje oczekiwany wynik biznesowy.
- Niepoprawny input zwraca kontrolowaną odpowiedź `4xx`.
- Process failures zwracają stabilny błąd aplikacji bez surowych szczegółów komendy.

### Testy składni shella

Input zawierający poniższe elementy nie może uruchamiać dodatkowych komend:

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

Dokładne wartości testowe powinny być bezpiecznymi markerami i pozostać w autoryzowanych środowiskach testowych.

### Testy timingowe

- Zapisz normalny zakres czasu odpowiedzi.
- Wyślij nieszkodliwy marker przypominający delay.
- Powtórz test.
- Zwiększ żądaną wartość opóźnienia.
- Potwierdź, że czas odpowiedzi nie skaluje się z markerem.

Timing tests powinny uwzględniać normalną zmienność CI i nie opierać się na jednym requeście.

### Testy side effectu w pliku

- Input nie może utworzyć nowego pliku w web root.
- Input nie może nadpisać ani dopisać do istniejącego pliku.
- Tymczasowy output jest zapisywany tylko do dedykowanego katalogu niedostępnego przez web.
- Konto aplikacji nie może zapisywać poza zatwierdzonymi katalogami.

### Testy network side effect

- Input nie może wywołać outbound DNS ani HTTP requests.
- Worker albo aplikacja ma tylko wymagany egress access.
- Środowiska testowe mogą używać kontrolowanego lokalnego listenera albo OAST, jeśli polityka na to pozwala.

## Code-review assertions

### Node.js

- Żadna wartość kontrolowana przez użytkownika nie trafia do `child_process.exec()`.
- `spawn()` i `execFile()` nie włączają `shell: true`.
- Argumenty są osobnymi elementami tablicy.
- Timeout i limity outputu są skonfigurowane.

### Python

- Żadna wartość kontrolowana przez użytkownika nie trafia do `os.system()`.
- `subprocess` używa listy argumentów.
- `shell=False` jest jawne albo zachowane.
- `check`, timeout, stdout i stderr handling są zdefiniowane.

### PHP

- User input nie jest doklejany do `exec()`, `system()`, `shell_exec()` ani `passthru()`.
- Native library albo API zastępuje shell execution tam, gdzie to możliwe.
- Escaping nie jest traktowany jako jedyna kontrola bezpieczeństwa.

### Java

- `ProcessBuilder` używa osobnych elementów listy.
- User input nie przechodzi przez `sh -c`, `bash -c` ani `cmd /c`.
- Komenda i flagi są kontrolowane po stronie serwera.

### .NET

- `UseShellExecute` jest false dla process execution z dynamicznymi argumentami.
- `ArgumentList` jest preferowane względem budowania stringa argumentów.
- Ścieżka executable jest stała i zaufana.

### Ruby

- Interpolowane backticks i `%x` nie są używane z user input.
- `Open3` albo `system` otrzymuje osobne wartości programu i argumentów.

### Go

- User input nie trafia do `sh -c`.
- `exec.CommandContext` używa stałego executable i osobnych argumentów.
- Context timeout i output handling są zdefiniowane.

## Testy uprawnień

- Potwierdź, że aplikacja działa jako dedykowane konto non-root albo non-administrator.
- Potwierdź, że konto nie może czytać niezwiązanych sekretów.
- Potwierdź, że konto nie może zapisywać do web root.
- Potwierdź, że niepotrzebne binaria są niedostępne albo blokowane.
- Potwierdź, że environment variables wystawione child processes są zminimalizowane.
- Potwierdź, że network egress odpowiada wymaganiom funkcji.

## Testy obserwowalności

- Odrzucone próby wykonania tworzą odpowiedni security log.
- Logi zawierają korelację requestu, ale wykluczają sekrety i pełne złośliwe payloady tam, gdzie nie są potrzebne.
- Nieoczekiwane child-process creation generuje alert w środowiskach wyższego ryzyka.
- Powtarzające się timing probes albo process failures są wykrywalne.
- Tworzenie plików w wrażliwych katalogach jest monitorowane.

## Przykładowe acceptance criteria

```text
Given feedback email value containing shell metacharacters
When the feedback endpoint processes the request
Then no shell process is started
And no additional child command is executed
And response time remains within the normal tolerance
And no file is created in a web-readable directory
And no outbound interaction occurs
And the request receives a controlled validation response or is processed as literal data
```

## Manualna weryfikacja po wdrożeniu

1. Potwierdź, że wdrożony code path odpowiada przejrzanej implementacji.
2. Powtórz oryginalny bezpieczny proof w autoryzowanym środowisku.
3. Porównaj timing z baseline.
4. Sprawdź poprzednio zapisywalną ścieżkę.
5. Sprawdź telemetry procesu i sieci.
6. Zweryfikuj konto aplikacji i uprawnienia.
7. Zapisz dowody, że podatność nie reprodukuje się.
