# A02 Laby i praktyka: Security Misconfiguration

## Ukończona praktyka

### TryHackMe

**Room/module:** OWASP Top 10 2025: Application Design Flaws  
**Task:** Task 2 - AS02: Security Misconfigurations  
**Status:** Ukończone  
**Główny pattern:** Verbose API error response ujawniający informacje debugowe i wrażliwe dane

## Kontekst laba

To było celowo podatne zadanie edukacyjne użyte do legalnej praktyki i dokumentacji.

Lab prezentował User Management API z udokumentowanym endpointem:

```http
GET /api/user/<user_id>
```

Przykład:

```http
GET /api/user/123
```

Strona mówiła, że user ID musi być numeryczne.

To było dobre ćwiczenie A02, bo problem nie polegał na ominięciu authentication ani zmianie ownership checks. Problem pojawiał się wtedy, gdy aplikacja dostawała nieoczekiwany input i zwracała niebezpieczne informacje debugowe.

## Co ćwiczyłem

W tym labie ćwiczyłem:

- czytanie dokumentacji API wystawionej przez aplikację,
- testowanie najpierw oczekiwanego inputu,
- testowanie invalid input po zrozumieniu oczekiwanego formatu,
- porównywanie normalnych odpowiedzi z error responses,
- identyfikowanie verbose error output,
- rozpoznawanie traceback disclosure,
- odróżnianie Security Misconfiguration od IDOR lub Broken Access Control,
- myślenie o bezpiecznej produkcyjnej obsłudze błędów,
- zamianę dowodów z laba w AppSec-style finding.

## Ważne rozróżnienie

Problemem nie było to, że użytkownik mógł uzyskać rekord innego użytkownika przez zmianę ID.

Problemem było to, że invalid input powodował ujawnienie debug details i wrażliwej wartości w odpowiedzi API.

To sprawia, że podatność jest Security Misconfiguration, a nie przede wszystkim IDOR albo Broken Access Control.

## Zaobserwowane zachowanie

### Normalny request numeryczny

Numeryczny user ID zwracał generyczny obiekt użytkownika:

```http
GET /api/user/123
```

Przykładowa odpowiedź:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

Zmiana numerycznych ID zwracała podobne dummy data. To sugerowało, że prosta enumeracja ID nie była głównym problemem w tym zadaniu.

### Niepoprawny request nienumeryczny

Wartość nienumeryczna taka jak `admin` wywoływała verbose error response:

```http
GET /api/user/admin
```

Odpowiedź zawierała informacje debugowe podobne do:

```json
{
  "debug_info": {
    "flag": "<redacted-lab-flag>",
    "error": "Invalid user ID format: admin. Flag: <redacted-lab-flag>",
    "traceback": "Traceback (most recent call last):\n  File \"/app/app.py\", line 21, in get_user\n    raise ValueError(...)\nValueError: Invalid user ID format: admin. Flag: <redacted-lab-flag>\n"
  }
}
```

Prawdziwa flaga z laba została celowo zredagowana w tych notatkach. Sekrety, flagi, tokeny i realne wrażliwe wartości nie powinny trafiać do repozytorium.

## Co było podatne

Podatnym zachowaniem była odpowiedź błędu zwracana dla invalid input:

```http
GET /api/user/admin
```

Odpowiedź ujawniała:

- wewnętrzną ścieżkę pliku: `/app/app.py`,
- nazwę funkcji: `get_user`,
- numer linii: `line 21`,
- typ wyjątku: `ValueError`,
- szczegóły traceback,
- wrażliwą wartość flagi labowej.

## Dlaczego to jest Security Misconfiguration

To jest Security Misconfiguration, bo aplikacja ujawniała verbose debug information i wrażliwe dane przez client-facing API error.

To nie jest głównie IDOR, bo problem nie dotyczył dostępu do obiektu innego użytkownika przez zmianę ID.

To nie jest głównie Broken Access Control, bo problem nie dotyczył brakujących authorization checks.

Root problemem była niebezpieczna produkcyjna obsługa błędów i debug information disclosure.

## Co mnie zmyliło lub było warte zauważenia

### Numeryczne ID zwracały tego samego użytkownika

Na początku zmiana numerycznych ID wyglądała jak oczywisty test, bo endpoint był `/api/user/<user_id>`. Jednak każde numeryczne ID zwracało tego samego dummy usera.

To była wskazówka, że zadanie prawdopodobnie nie dotyczy IDOR ani Broken Access Control.

### Fraza "User ID must be numeric" była wskazówką

Ważnym testem nie było tylko próbowanie innych numerycznych ID. Lepszy test A02 brzmiał:

> Co się stanie, gdy aplikacja dostanie input łamiący oczekiwany format?

Podanie wartości nienumerycznej wywołało verbose error response.

### 400 vs 404

Dla requestu takiego jak:

```http
GET /api/user/admin
```

bezpieczna odpowiedź powinna zwykle wyglądać tak:

```http
400 Bad Request
```

bo endpoint istnieje, ale format inputu jest niepoprawny.

`404 Not Found` może być użyte w niektórych projektach, ale dla błędów walidacji inputu `400 Bad Request` jest zwykle czytelniejsze.

## Jak testowałbym to w prawdziwej aplikacji

Szukałbym tego patternu w:

- publicznych endpointach API,
- route'ach user lookup,
- parametrach search i filter,
- stronach admin i status,
- endpointach health lub debug,
- przetwarzaniu uploadów,
- JSON APIs,
- route'ach opartych o CMS,
- stronach błędów reverse proxy i web servera,
- deploymentach staging lub preview.

Dla każdego obszaru testowałbym:

- valid input jako baseline,
- brakujący input,
- niepoprawne typy,
- bardzo długie wartości,
- znaki specjalne,
- nieoczekiwane metody HTTP,
- bezpośredni dostęp do typowych ścieżek debug/config,
- czy błędy zawierają traceback, ścieżki plików, sekrety lub szczegóły frameworka.

## Review result

Praktyczna lekcja z tego laba jest prosta: invalid input jest normalny, ale debug output nie może stać się częścią publicznego API.

Z nietechnicznej perspektywy to jak pokazanie klientowi wewnętrznej instrukcji serwisowej, notatek pracowników i sekretnych kodów magazynowych za każdym razem, gdy wpisze niepoprawny numer konta.

Główne takeaways:

- endpoint działał dla oczekiwanego inputu numerycznego,
- niebezpieczne zachowanie pojawiało się dopiero po złamaniu formatu inputu,
- API zwracało implementation details, które powinny zostać w logach server-side,
- wrażliwe wartości nigdy nie powinny być częścią exception messages,
- dobra poprawka wymaga testów sprawdzających zarówno bezpieczny status błędu, jak i brak debug strings.

## Powiązane notatki wewnętrzne

- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite Proxy i Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)

## Pomysły na przyszłą praktykę

Możliwa kolejna praktyka związana z A02 obejmuje:

- zreviewować lokalną aplikację pod debug mode i verbose error responses,
- sprawdzić security headers i CORS na deploymentcie testowym,
- wykonać content discovery dla wystawionych `.env`, backupów, logów lub ścieżek debug w legalnym labie,
- porównać konfigurację development vs production w przykładowym projekcie,
- napisać testy regresji dla generycznej obsługi błędów API.
