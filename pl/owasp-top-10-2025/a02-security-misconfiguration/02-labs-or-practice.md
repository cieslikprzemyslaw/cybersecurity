# A02 Laby i praktyka: Security Misconfiguration

## Ukończona praktyka

### TryHackMe

**Room/module:** OWASP Top 10 2025: Application Design Flaws  
**Task:** Task 2 - AS02: Security Misconfigurations  
**Status:** Ukończone  
**Główny pattern:** Verbose API error response ujawniający informacje debugowe i wrażliwe dane

Link zewnętrzny:

- https://tryhackme.com/room/owasptopten2025two

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

## Review result

Praktyczna lekcja z tego laba jest prosta: invalid input jest normalny, ale debug output nie może stać się częścią publicznego API.

Główne takeaways:

- endpoint działał dla oczekiwanego inputu numerycznego,
- niebezpieczne zachowanie pojawiało się dopiero po złamaniu formatu inputu,
- API zwracało implementation details, które powinny zostać w logach server-side,
- wrażliwe wartości nigdy nie powinny być częścią exception messages,
- dobra poprawka wymaga testów sprawdzających zarówno bezpieczny status błędu, jak i brak debug strings.

---

### Lab 2: PortSwigger - Information Disclosure on Debug Page

**Platforma:** PortSwigger Web Security Academy
**Lab:** Information disclosure on debug page
**Status:** Ukończone
**Główny pattern:** Publiczny debug endpoint ujawniający konfigurację i sekret aplikacyjny

Link zewnętrzny:

- https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-on-debug-page

## Kontekst laba

W tym labie aplikacja nie pokazywała debug linku w UI, ale link był obecny w HTML source jako komentarz:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

To był dobry przykład A02, bo problem nie polegał na samym komentarzu. Komentarz był tylko wskazówką discovery. Prawdziwym problemem było to, że backendowy debug endpoint nadal istniał i był publicznie dostępny.

## Co ćwiczyłem

W tym labie ćwiczyłem:

- sprawdzanie HTML source jako części reconu,
- rozpoznawanie ukrytych lub pozostawionych debug links,
- bezpośredni dostęp do odkrytego endpointu,
- ocenę ryzyka publicznego `phpinfo()`,
- rozróżnienie między discoverability a właściwą podatnością,
- opisanie wpływu ujawnionego sekretu,
- zaplanowanie remediacji i testów regresji.

## Zaobserwowane zachowanie

Po wejściu bezpośrednio na endpoint:

```http
GET /cgi-bin/phpinfo.php
```

aplikacja zwracała publiczną stronę `phpinfo()`.

Strona ujawniała szczegóły konfiguracji PHP i serwera, między innymi:

- wersję PHP,
- szczegóły systemu operacyjnego,
- Server API,
- ścieżkę do głównego pliku konfiguracyjnego PHP,
- dodatkowe pliki `.ini`,
- załadowane moduły i rozszerzenia,
- ustawienia PHP,
- dane środowiskowe/konfiguracyjne,
- wrażliwy `SECRET_KEY`.

Prawdziwy sekret został celowo zredagowany:

```text
<redacted-secret-key>
```

## Co było podatne

Podatne było publiczne wystawienie debug endpointu:

```http
GET /cgi-bin/phpinfo.php
```

Komentarz HTML ułatwił znalezienie endpointu, ale nie był właściwym zabezpieczeniem ani główną podatnością.

## Dlaczego to jest Security Misconfiguration

To jest Security Misconfiguration, bo funkcjonalność diagnostyczna/developerska została wdrożona lub pozostawiona w publicznym środowisku.

W produkcji `phpinfo()` nie powinno być publicznie dostępne. Taka strona może ujawnić wystarczająco dużo informacji, żeby ułatwić fingerprinting, ukierunkowane testy kolejnych podatności lub wykorzystanie ujawnionego sekretu.

## Bezpieczne zachowanie

Najlepszy wynik dla produkcji:

```http
HTTP/1.1 404 Not Found
```

bo debug file nie powinien być w ogóle wdrożony.

Jeśli diagnostyka jest naprawdę potrzebna, powinna być dostępna tylko dla zaufanego/admin/internal access. Wtedy użytkownik bez uprawnień powinien dostać:

```http
HTTP/1.1 403 Forbidden
```

lub challenge uwierzytelnienia, zależnie od projektu aplikacji.

## Ważna uwaga remediacyjna

Usunięcie strony nie wystarcza, jeśli sekret został ujawniony.

Należy:

- usunąć lub zablokować debug endpoint,
- usunąć debug reference z HTML/templates/frontend assets,
- obrócić ujawniony `SECRET_KEY`,
- sprawdzić logi, artifacty deploymentu i historię repozytorium,
- dodać secret scanning i deployment checks.

## Najważniejsze wnioski

Dwa praktyczne patterny A02 z tych labów:

1. invalid input powodujący verbose debug/traceback disclosure,
2. publiczny debug endpoint ujawniający `phpinfo()` i sekret aplikacyjny.

Dłuższe refleksje i real-world review angle są w:

- [A02 learning notes](05-learning-notes.md)
- [A02 overview](01-overview.md)
- [A02 checklista](03-checklist.md)
- [A02 testy regresji](04-regression-tests.md)
- [Verbose API error finding](security-findings/01-example-finding.md)
- [Public debug endpoint finding](security-findings/02-public-debug-endpoint-phpinfo.md)

## Powiązane notatki wewnętrzne

- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite Proxy i Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
