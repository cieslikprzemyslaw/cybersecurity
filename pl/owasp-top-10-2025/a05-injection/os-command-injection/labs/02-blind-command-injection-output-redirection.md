# Lab 2 - Blind OS Command Injection z Output Redirection

## Platforma

PortSwigger Web Security Academy

Lab: https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection

## Zakres

Wyłącznie autoryzowany lab treningowy.

## Cel labu

Zidentyfikować użytkownika systemowego procesu aplikacji, gdy output komendy nie wraca w odpowiedzi podatnego endpointu.

## Podatna funkcja

```text
POST /feedback/submit
```

Normalne wartości formularza:

```text
csrf=<token>
name=name
email=email@gmail.com
subject=com
message=message
```

## Odkrycie requestu

Najpierw widoczne były dwa requesty:

```http
GET /product?productId=2
GET /image?filename=44.jpg
```

Request obrazka był ciekawy, bo lab dotyczył output redirection, ale nie był sinkiem feedback submission.

Właściwy request znaleziono po wysłaniu formularza feedbacku i sprawdzeniu Burp HTTP history:

```http
POST /feedback/submit
```

## Kontrolowane parametry

Formularz pozwalał kontrolować:

- `name`,
- `email`,
- `subject`,
- `message`.

CSRF token był wymagany, ale nie był podejrzanym inputem komendy.

## Pierwsze założenie

Najpierw wybrałem `message`, bo spodziewałem się słabszej walidacji free-text field.

To rozumowanie było niepełne. Słaba walidacja nie tworzy Command Injection, jeśli wartość nie trafia do komendy albo process sink.

## Lepsza hipoteza

Pole `email` było bardziej prawdopodobne, bo funkcja feedbacku mogła przekazywać adres email do procesu związanego z mailem.

To nadal była tylko hipoteza. Dokładna komenda backendowa nie była widoczna.

## Normalny baseline

Normalne requesty feedbacku kończyły się w milisekundach.

## Pierwsza próba timingowa

Nieszkodliwy loopback ping został doklejony do wartości email z jednym separatorem na początku.

Wynik:

```text
84 ms
500 Internal Server Error
"Could not save"
```

To nie dowodziło wykonania.

Możliwe wyjaśnienia:

- błąd składni komendy,
- stały suffix stał się argumentem wstrzykniętej komendy,
- błąd walidacji albo aplikacji,
- błędny kontekst komendy.

## Izolowanie komendy

Separator dodano po wstrzykniętej komendzie oraz przed nią.

Struktura koncepcyjna:

```text
oryginalna komenda ; ping ... ; oryginalny suffix
```

Separator zamykający prawdopodobnie zapobiegł temu, żeby suffix backendu stał się częścią wstrzykniętej komendy.

## Dowód timingowy

Zaobserwowane wyniki:

```text
ping -c 5  -> 4,128 ms
ping -c 10 -> 9,256 ms
```

Dlaczego to był mocny dowód:

- normalne requesty kończyły się w milisekundach,
- opóźnienie się powtarzało,
- opóźnienie rosło wraz z kontrolowaną liczbą pakietów,
- body i status odpowiedzi mogły pozostać takie same, mimo że czas wykonania się zmienił.

Odpowiedź `500` była wskazówką błędu aplikacji, nie dowodem. Dowodem command execution było proporcjonalne opóźnienie.

## Dlaczego to było blind

Odpowiedź z podatnego feedback endpointu nie zawierała stdout wstrzykniętej komendy.

Aplikacja zwracała tylko:

```text
"Could not save"
```

Wykonanie trzeba było więc pokazać przez side effects.

## Plan output redirection

Lab udostępniał endpoint obrazków:

```http
GET /image?filename=44.jpg
```

Udokumentowany zapisywalny katalog:

```text
/var/www/images/
```

Wynik nieszkodliwej komendy został przekierowany do:

```text
/var/www/images/output.txt
```

Fragment shella:

```text
whoami > /var/www/images/output.txt
```

`>` przekierowuje standard output. Nie jest samym outputem.

## Pobranie wyniku

Utworzony plik pobrano przez:

```http
GET /image?filename=output.txt
```

Odpowiedź:

```text
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8

peter-VqYsRp
```

## Łańcuch dowodowy

```text
kontrolowany email
-> separatory shella zmieniły strukturę komendy
-> powtarzalny timing potwierdził ukryte wykonanie
-> whoami wykonało się
-> stdout przekierowano do /var/www/images/output.txt
-> endpoint /image pobrał utworzony plik
-> zidentyfikowano użytkownika procesu aplikacji
```

## Ważne rozróżnienia

- `POST /feedback/submit` był sinkiem command injection.
- `email` był potwierdzonym parametrem kontrolowanym.
- `/image` nie wykonywał komendy.
- `/image` był osobnym kanałem pobrania wyniku.
- Dokładna oryginalna komenda backendowa nie była znana.
- Udany Unix-style separator i komenda sugerowały Unix-like shell context, ale nie dowodziły dokładnego shella.

## Przyczyna źródłowa

Niezaufany input email mógł wpłynąć na string komendy interpretowany przez shell. Aplikacja nie oddzieliła danych użytkownika od wykonywalnej składni komendy.

## Realny impact

Proof pokazał arbitrary command execution jako konto aplikacji. Zależnie od uprawnień atakujący mógłby czytać pliki, modyfikować zapisywalne treści, uzyskać sekrety, wywoływać narzędzia, zakłócać aplikację albo używać dostępnego network access.

## Remediacja

- Zastąp shell-based feedback processing biblioteką email, SDK, service API albo kolejką.
- Jeśli zewnętrzny proces jest nieunikniony, użyj stałego executable i osobnych argumentów z wyłączonym shellem.
- Hardcode command flags i odrzucaj option-like input.
- Stosuj ścisłą walidację email jako kontrolę drugorzędną, nie główną poprawkę.
- Uruchamiaj jako dedykowane konto o niskich uprawnieniach.
- Usuń write access do web-readable directories, jeśli nie jest jawnie wymagany.
- Dodaj timeouty, limity outputu i monitoring.
- Dodaj testy regresji behawioralne, code-level, filesystem i egress.

## Błędy i korekty z nauki

- Najpierw wybrałem `message` na podstawie założeń o walidacji, a nie prawdopodobnego data flow.
- Początkowo opisałem użycie emaila jako marketingowy fakt, ale cel nie był potwierdzony.
- Pierwszy test użył tylko jednego separatora i szybko zwrócił `500`.
- Nauczyłem się, że separator zamykający może izolować wstrzykniętą komendę od suffixu backendu.
- Nauczyłem się nie traktować jednego `500` jako dowodu sukcesu albo porażki.
- Potwierdziłem wykonanie przez powtarzalny i proporcjonalny timing.
- Początkowo nazwałem `>` outputem i poprawiłem to na operator przekierowania stdout.
- Pierwsza remediacja zbyt mocno opierała się na walidacji i sanitizacji.
- Poprawiony model remediacji priorytetyzuje unikanie shella i bezpieczne process APIs z osobnymi argumentami.

## Status wiedzy po debriefie

Lab został ukończony. Debrief początkowo dał **PARTIAL PASS**, bo root cause i remediation nie były jeszcze wyjaśnione wystarczająco mocno. Te notatki zawierają poprawiony model.

## Testy regresji

- Wartości `email` zawierające separatory nie mogą uruchamiać dodatkowych komend.
- Marker opóźnienia nie zwiększa czasu odpowiedzi.
- Zwiększenie wartości markera nie tworzy proporcjonalnego opóźnienia.
- Input nie może tworzyć pliku w `/var/www/images/`.
- Konto aplikacji nie może zapisywać do web-readable directory.
- Funkcja nie buduje stringa komendy shella.
- Jeśli process execution pozostaje konieczne, używany jest stały executable i osobna lista argumentów.
- Argumenty zaczynające się od `-` nie mogą stać się opcjami.
- Formularz nie może wywołać outbound interaction.
