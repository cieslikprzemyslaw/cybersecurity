# Blind OS Command Injection w formularzu feedbacku

## Podsumowanie

Endpoint wysyłki feedbacku przekazywał wartość email kontrolowaną przez użytkownika do komendy shella. Metaznaki shella w parametrze `email` zmieniały wykonanie komend systemowych.

Aplikacja nie zwracała outputu komendy bezpośrednio, ale wykonanie potwierdzono przez powtarzalny timing oraz przekierowanie outputu do pliku dostępnego przez web.

## Sugerowana istotność

**High**

Dokładna istotność zależy od uprawnień procesu aplikacji, dostępnych plików, dostępu sieciowego, dostępnych binariów i otaczającej infrastruktury. Arbitrary command execution nawet jako niskouprawniony użytkownik web może ujawnić dane aplikacji i stworzyć ścieżkę do szerszego kompromisu.

## Komponent

```text
POST /feedback/submit
Parameter: email
```

## Podatne zachowanie

Endpoint przyjmował wartość email i używał jej jako części komendy shella. Separator komend został zinterpretowany jako składnia shella, a nie zwykłe dane email.

Koncepcyjny przepływ:

```text
email kontrolowany przez użytkownika
-> backend buduje string komendy
-> shell interpretuje separatory
-> dodatkowa komenda OS wykonuje się
```

Dokładna oryginalna komenda backendowa nie była widoczna. Prawdopodobnie dotyczyła obsługi albo wysyłki feedbacku, ale to pozostaje wnioskiem, nie potwierdzonym faktem.

## Dowody

### Dowód timingowy

Normalny request kończył się w milisekundach.

Kontrolowany test z loopback `ping` dawał około:

```text
ping count 5  -> około 4,1 sekundy
ping count 10 -> około 9,3 sekundy
```

Ten sam endpoint zwracał `500 Internal Server Error` z `"Could not save"`, ale opóźnienie skalowało się z kontrolowaną liczbą pakietów. Powtarzalny i proporcjonalny timing był mocnym dowodem wykonania komendy.

### Dowód plikowy

Output `whoami` został przekierowany do zapisywalnego katalogu obrazków:

```text
/var/www/images/output.txt
```

Plik pobrano potem przez:

```http
GET /image?filename=output.txt
```

Odpowiedź zwróciła:

```text
peter-VqYsRp
```

Losowy suffix był specyficzny dla instancji labu. Wynik pokazał, że wstrzyknięta komenda wykonała się jako użytkownik procesu aplikacji.

## Dlaczego podatność była blind

Odpowiedź z `POST /feedback/submit` nie zawierała outputu komendy. Dowody uzyskano przez side effects:

- timing odpowiedzi,
- utworzenie pliku,
- osobny request pobierający plik.

Endpoint `/image` nie był sinkiem command execution. Tylko udostępnił plik utworzony przez wstrzykniętą komendę.

## Przyczyna źródłowa

Niezaufany input mógł wpłynąć na string komendy shella. Aplikacja nie utrzymała granicy bezpieczeństwa między danymi a wykonywalną składnią shella.

Root cause nie brzmi tylko „brak sanitizacji”. Niebezpieczny design polegał na użyciu inputu użytkownika w konstrukcji komendy interpretowanej przez shell.

## Impact

Zależnie od uprawnień procesu i kontroli środowiska atakujący mógłby potencjalnie:

- czytać pliki dostępne dla aplikacji,
- modyfikować zapisywalne pliki,
- uzyskać sekrety z konfiguracji albo environment data,
- wywoływać zainstalowane narzędzia,
- wykonywać outbound network requests,
- zakłócić dostępność aplikacji,
- użyć przejętego procesu jako punktu startowego do dalszych ataków.

Do potwierdzenia problemu w autoryzowanym labie nie były potrzebne destrukcyjne komendy.

## Remediacja

1. Usuń wywołanie komendy OS, jeśli biblioteka języka, email SDK, kolejka albo service API może wykonać zadanie.
2. Jeśli process execution jest nieuniknione, użyj stałego executable i przekaż każdy argument osobno.
3. Nie wywołuj shella i nie buduj jednego stringa komendy przez konkatenację.
4. Hardcode wymagane flagi i mapuj ograniczone wybory użytkownika na allowlistowane wartości po stronie serwera.
5. Zapobiegaj argument injection, w tym wartościom wyglądającym jak opcje i zaczynającym się od `-`.
6. Uruchamiaj aplikację jako dedykowane konto o niskich uprawnieniach.
7. Ogranicz write access do filesystemu, szczególnie katalogów web-readable.
8. Zastosuj timeouty, limity outputu i resource controls.
9. Loguj i alertuj nieoczekiwane child processes, powtarzające się błędy wykonania, podejrzane opóźnienia i nietypowe tworzenie plików.

## Testy regresji

- Separatory shella w `email` muszą być odrzucane albo traktowane jako literalne dane.
- Marker testowy nie może powodować mierzalnego opóźnienia wykonania.
- Zwiększenie wartości opóźnienia nie może proporcjonalnie zwiększać czasu odpowiedzi.
- Input nie może tworzyć plików w `/var/www/images/` ani innym katalogu aplikacji.
- Input nie może wywołać outbound DNS albo HTTP interactions.
- Code review musi potwierdzić, że funkcja nie używa stringa komendy shella.
- Jeśli process execution nadal jest wymagane, musi używać stałego executable i osobnych argumentów z wyłączonym shellem.
- Konto aplikacji nie może mieć niepotrzebnych uprawnień read, write ani execute.

## Autoryzowane odtworzenie - skrót

1. Przechwyć normalny feedback submission.
2. Ustal baseline czasu odpowiedzi.
3. Zmień tylko podejrzany parametr, używając nieszkodliwego timing test.
4. Powtórz test i zmień kontrolowane opóźnienie, aby potwierdzić proporcję.
5. Przekieruj wynik nieszkodliwej komendy do udokumentowanego zapisywalnego katalogu labu.
6. Pobierz plik przez endpoint obrazków labu.
7. Zapisz odpowiedź, timing, output, założenia i dokładną przyczynę źródłową.

## Referencje

- PortSwigger OS Command Injection: https://portswigger.net/web-security/os-command-injection
- PortSwigger blind output redirection lab: https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection
- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
