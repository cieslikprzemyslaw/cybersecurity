# OS Command Injection - Remediacja

## Priorytet remediacji

Preferowana kolejność:

1. unikaj OS command execution,
2. unikaj shella,
3. używaj stałego executable i osobnych argumentów,
4. waliduj i allowlistuj wartości,
5. zapobiegaj argument injection,
6. ogranicz uprawnienia i dostępne zasoby,
7. stosuj limity wykonania,
8. loguj, monitoruj i dodaj testy regresji.

## 1. Unikaj komend OS

Najmocniejsza poprawka to zastąpienie komendy przez:

- funkcję standard library,
- utrzymywaną bibliotekę aplikacyjną,
- SDK,
- service API,
- kolejkę albo worker z wąskim interfejsem.

Przykłady:

- użyj filesystem API zamiast `system("mkdir ...")`,
- użyj biblioteki email albo provider API zamiast budować shellową komendę `mail`,
- użyj bindingu do image-processing library zamiast sklejania komendy CLI,
- odpytuj strukturalne dane zamiast używać `grep` przez shell.

To usuwa granicę interpretacji shella zamiast próbować bronić jej coraz bardziej złożonym czyszczeniem stringów.

## 2. Unikaj shella

Jeśli zewnętrzny executable jest konieczny:

- nie używaj `shell=True`,
- nie używaj `sh -c`, `bash -c`, `cmd /c` ani stringów PowerShella,
- nie używaj API wymagających jednego interpolowanego stringa komendy,
- używaj process API przyjmującego executable i listę argumentów osobno.

Docelowy wzorzec:

```text
stały executable
+ strukturalne argumenty
+ shell disabled
```

## 3. Oddziel executable i argumenty

Unsafe:

```js
exec(`convert ${source} ${destination}`);
```

Bezpieczniejszy kształt:

```js
execFile("convert", [source, destination], options, callback);
```

To zapobiega interpretacji separatorów przez shell. Nie sprawia automatycznie, że każdy argument jest bezpieczny.

## 4. Używaj ścisłych allowlist

Gdy funkcja wspiera ograniczony zestaw wartości, mapuj wybory klienta na stałe po stronie serwera.

Przykład:

```text
client sends: formatId=webp
server maps:  webp -> approved internal format value
```

Nie akceptuj:

- dowolnych nazw executable,
- dowolnych flag,
- fragmentów komend,
- free-form pipeline definitions,
- working directories kontrolowanych przez użytkownika.

Walidacja powinna definiować:

- akceptowany typ,
- dokładne dozwolone wartości albo zestaw znaków,
- minimalną i maksymalną długość,
- granice liczbowe,
- reguły nazw plików,
- czy whitespace jest dozwolony.

## 5. Zapobiegaj argument injection

Proces może być niebezpieczny nawet bez shella, jeśli input atakującego staje się opcją.

Przykład:

```text
oczekiwany operand: filename
wartość atakującego: --unexpected-option
```

Kontrole:

- użyj `--` do zakończenia parsowania opcji tam, gdzie program to wspiera,
- hardcode wymagane flagi,
- odrzucaj wartości wyglądające jak opcje, jeśli nie są poprawne,
- waliduj zgodnie z gramatyką argumentów docelowego programu,
- nie wystawiaj surowych funkcji CLI przez HTTP API.

## 6. Least privilege

Proces aplikacji powinien:

- działać jako dedykowane konto bez uprawnień administratora,
- czytać tylko wymagane pliki,
- zapisywać tylko wymagane katalogi,
- unikać zapisu do web root,
- mieć minimalny dostęp sieciowy,
- wykonywać tylko potrzebne binaria,
- otrzymywać minimalne środowisko.

Least privilege zmniejsza impact. Nie naprawia unsafe command construction.

## 7. Timeouty i limity zasobów

Process execution powinno definiować:

- timeout wykonania,
- maksymalny rozmiar stdout i stderr,
- limity pamięci i CPU tam, gdzie są dostępne,
- limity współbieżności,
- kontrolowany working directory,
- zachowanie cancellation,
- bezpieczne czyszczenie plików tymczasowych.

## 8. Bezpieczna obsługa outputu i błędów

- Nie zwracaj użytkownikom surowych błędów procesu ani stringów komend.
- Nie zapisuj sekretów w logach.
- Traktuj stdout i stderr jako niezaufane dane.
- Nie renderuj outputu komendy jako HTML bez właściwego output encoding.
- Używaj kontrolowanych błędów aplikacji.
- Zapisuj wewnętrzny kontekst potrzebny do analizy bez ujawniania go zewnętrznie.

## 9. Logging i monitoring

Monitoruj:

- powtarzające się process failures,
- metaznaki shella w polach, które nie powinny ich zawierać,
- nietypowy czas wykonania,
- nieoczekiwane child processes,
- zapisy do nietypowych katalogów,
- outbound DNS albo HTTP activity,
- skoki liczby uruchamianych procesów,
- odrzucone próby wykonania.

Logging wspiera detection i response, ale nie jest główną kontrolą prewencyjną.

## 10. Testy regresji

Kompletna poprawka powinna potwierdzić, że:

- shell nie jest już wywoływany,
- executable selection jest kontrolowany po stronie serwera,
- argumenty są osobne i zwalidowane,
- separatory nie tworzą dodatkowych komend,
- option-like input nie zmienia zachowania programu,
- testy timingowe nie tworzą kontrolowanych opóźnień,
- nieoczekiwane pliki nie są tworzone,
- outbound interactions nie występują,
- uprawnienia procesu i dostęp do filesystemu są ograniczone.

## Dlaczego sama sanitizacja jest słaba

Shell parsing różni się między:

- systemami operacyjnymi,
- shellami,
- kontekstami quoting,
- encodingiem,
- command substitutions,
- environment expansion,
- warstwami decode aplikacji.

Blacklisty niebezpiecznych znaków łatwo zrobić niekompletne. Escaping może też zatrzymać shell injection, ale zostawić argument injection.

Dlatego:

```text
avoid shell > structured process API > allowlist > escaping jako wsparcie ostatniej szansy
```

## Przykład developerskiego review

Słaba remediacja:

> Usunęliśmy średniki z wartości email.

Mocniejsza remediacja:

> Funkcja feedbacku nie buduje już komendy shella. Używa SDK dostawcy email. Adres docelowy jest skonfigurowany po stronie serwera, email użytkownika jest walidowany jako wartość email, aplikacja działa na ograniczonym koncie, a testy regresji potwierdzają, że metaznaki nie powodują timingu, plików ani network side effects.

## Referencje

- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- PortSwigger prevention guidance: https://portswigger.net/web-security/os-command-injection
