# A02 Checklista: Security Misconfiguration

Używaj tej checklisty podczas code review, API review, manual testing, review deploymentu frontendowego albo AppSec review.

Główne pytanie:

> Czy aplikacja ujawnia niebezpieczne zachowanie, bo konfiguracja, obsługa błędów albo hardening platformy są zbyt otwarte, zbyt gadatliwe albo zbyt podobne do development mode?

## 1. Zrozum powierzchnię

Przed testowaniem ustal:

- Jaka aplikacja, API lub usługa jest wystawiona?
- Czy to produkcja, staging, preview czy lokalny lab?
- Jaki framework, serwer, CMS, kontener lub cloud service bierze udział?
- Które endpointy są publiczne?
- Które endpointy powinny być internal-only?
- Jakiego formatu błędów używa aplikacja?
- Czy istnieją endpointy admin, debug, status, health lub config?
- Czy frontend bundles, source maps lub static assets są publicznie dostępne?

Przykłady wrażliwych obszarów konfiguracji:

- error handling,
- debug mode,
- security headers,
- CORS,
- cookies,
- source maps,
- admin routes,
- cloud storage,
- default accounts,
- exposed files,
- niepotrzebne usługi lub pluginy.

## 2. Error handling i debug output

Sprawdź:

- Czy debug lub development settings są wyłączone na produkcji?
- Czy error responses są generyczne i bezpieczne dla użytkowników?
- Czy szczegóły techniczne są logowane tylko server-side?
- Czy API zwraca stack trace'y?
- Czy API ujawnia wewnętrzne ścieżki plików?
- Czy API ujawnia nazwy funkcji, klas, numery linii lub szczegóły frameworka?
- Czy API ujawnia typy wyjątków, takie jak `ValueError`, `TypeError`, `NullReferenceException` lub database errors?
- Czy API ujawnia zapytania SQL, nazwy schematów, nazwy tabel lub connection details?
- Czy API ujawnia sekrety, tokeny, flagi, klucze API lub zmienne środowiskowe?
- Czy API zwraca spójne i bezpieczne formaty błędów?

## 3. Pytania do code review

Zapytaj:

- Gdzie zdefiniowana jest konfiguracja produkcyjna?
- Czy istnieje jasne rozdzielenie ustawień development, staging i production?
- Czy debug mode jest kontrolowany przez zmienną środowiskową lub deployment profile?
- Czy istnieje global error handler?
- Czy wyjątki są zamieniane na generyczne odpowiedzi dla klienta?
- Czy sekrety kiedykolwiek trafiają do exception messages?
- Czy surowe błędy backendowe są przekazywane bezpośrednio do frontendu?
- Czy domyślne ustawienia frameworka są reviewowane przed deploymentem?
- Czy niepotrzebne route'y, pluginy lub usługi są wyłączone?
- Czy hardening checks są częścią deploymentu lub CI?

## 4. Pytania testowe

Dla każdego ważnego endpointu testuj:

- Co dzieje się dla valid input?
- Co dzieje się, gdy brakuje wymaganych parametrów?
- Co dzieje się, gdy parametry mają zły typ?
- Co dzieje się, gdy ID są nienumeryczne, zbyt długie, ujemne, dziesiętne lub malformed?
- Co dzieje się dla znaków specjalnych?
- Co dzieje się dla nieoczekiwanych metod HTTP?
- Co dzieje się przy próbie dostępu do typowych ścieżek debug/config?
- Czy odpowiedź ujawnia szczegóły wewnętrzne?
- Czy odpowiedź ujawnia wrażliwe wartości?
- Czy odpowiedź pozostaje bezpieczna dla przypadków `400`, `401`, `403`, `404` i `500`?

Przykładowe testy invalid input:

```http
GET /api/user/admin
GET /api/user/abc
GET /api/user/-1
GET /api/user/1.5
GET /api/user/%27
GET /api/user/999999999999999999999999
```

Szukaj:

- `Traceback`,
- `Exception`,
- `ValueError`,
- `TypeError`,
- ścieżek plików takich jak `/app/...`, `/var/www/...`, `C:\...`,
- nazw frameworków,
- stack trace'ów,
- database errors,
- sekretów,
- tokenów,
- kluczy API,
- nazw lub wartości zmiennych środowiskowych.

## 5. Frontend-specific checks

Frontend review może szybko ujawnić symptomy misconfiguration.

Sprawdź:

- Czy source maps są wystawione na produkcji bez jasnego powodu?
- Czy debug flags lub development messages są widoczne w konsoli?
- Czy błędy backendowe są wyświetlane bezpośrednio w UI?
- Czy API errors są renderowane surowo użytkownikowi?
- Czy w JavaScript bundles są wrażliwe wartości?
- Czy assety frontendowe zawierają wewnętrzne endpointy?
- Czy client bundle ujawnia staging, preview lub internal URLs?
- Czy komentarze w HTML lub JavaScript ujawniają ukryte route'y, credentials, tickety lub internal notes?
- Czy frontend bezpiecznie obsługuje odpowiedzi `400`, `401`, `403` i `500`?

Ale potwierdź też:

- Odpowiedź backendu jest bezpieczna nawet bez filtrowania po stronie frontendu.
- Surowe wyjątki backendowe nie są częścią publicznego kontraktu API.
- Sekrety nie są osadzone w kodzie client-side.

## 6. Header i platform checks

Sprawdź nagłówki odpowiedzi i zachowanie platformy:

- Czy nagłówki bezpieczeństwa są obecne i odpowiednie?
- Czy `Content-Security-Policy` jest skonfigurowane tam, gdzie to praktyczne?
- Czy `X-Content-Type-Options` jest obecny?
- Czy `Referrer-Policy` jest obecny?
- Czy `Permissions-Policy` jest obecny tam, gdzie jest użyteczny?
- Czy `Strict-Transport-Security` jest włączone dla produkcyjnych stron HTTPS?
- Czy ochrona przed clickjackingiem jest skonfigurowana przez `frame-ancestors` lub `X-Frame-Options`?
- Czy wersje serwera/frameworka są niepotrzebnie ujawniane?
- Czy CORS jest skonfigurowany bezpiecznie?
- Czy cookies mają `HttpOnly`, `Secure` i `SameSite` tam, gdzie to właściwe?
- Czy directory listing jest wyłączony?
- Czy backupy, logi, config lub pliki tymczasowe są zablokowane przed publicznym dostępem?
- Czy endpointy admin/debug/status są chronione?

## 7. Developer red flags

Red flags:

- `debug=True` na produkcji,
- development error pages wystawione publicznie,
- exception messages zwracane bezpośrednio klientom API,
- `res.send(error)` lub równoważne patterny,
- surowe backend error objects przekazywane do frontendu,
- sekrety osadzone w exception messages,
- zmienne środowiskowe drukowane w odpowiedziach,
- zbyt permisywny CORS, na przykład odbijanie dowolnego Origin,
- publiczne pliki `.env`, `.git`, backup, log lub config,
- niechronione endpointy `/debug`, `/config`, `/admin`, `/status`, `/health`, `/actuator` lub `/server-status`,
- brak separacji środowisk,
- "To tylko staging, więc nie potrzebuje hardeningu."

## 8. Remediation checklist

Bezpieczniejsza implementacja powinna:

- wyłączyć debug mode na produkcji,
- dodać global error handler,
- zwracać generyczne komunikaty błędów dla klienta,
- logować szczegółowe błędy tylko server-side,
- usunąć sekrety z exception messages,
- walidować input przed uruchomieniem logiki biznesowej,
- standaryzować API error responses,
- oddzielić konfigurację produkcyjną od developerskiej,
- blokować dostęp do endpointów debug/config/internal,
- wyłączyć directory listing,
- usunąć domyślne credentials,
- usunąć niepotrzebne usługi, pluginy, route'y i strony testowe,
- utwardzić nagłówki bezpieczeństwa,
- zreviewować ustawienia CORS,
- dodać automatyczne testy regresji dla error disclosure.

## 9. Safe implementation notes

Dla invalid input preferuj kontrolowaną odpowiedź:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid user ID"
}
```

Nie zwracaj:

```text
Traceback
File "/app/app.py"
ValueError
DATABASE_URL
API_KEY
JWT_SECRET
```

Szczegółowe informacje powinny trafiać do logów server-side, monitoringu albo error trackingu, a nie do użytkownika.

## 10. Review outcome

Funkcja powinna przejść review A02 tylko wtedy, gdy:

- produkcyjne debug behaviour jest wyłączone,
- invalid input zwraca kontrolowane błędy,
- client-facing responses nie ujawniają stack trace'ów, ścieżek, typów wyjątków ani sekretów,
- niepotrzebne endpointy i pliki nie są publicznie dostępne,
- headers, CORS i cookies są skonfigurowane świadomie,
- frontend nie ujawnia sekretów ani raw debug output,
- istnieją testy regresji dla zaobserwowanej klasy misconfiguration.
