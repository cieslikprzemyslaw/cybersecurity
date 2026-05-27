# Przykładowe znalezisko: Publiczny debug endpoint ujawnia konfigurację aplikacji i secret key

## Podsumowanie

Publiczny debug endpoint ujawniał szczegółowe informacje o konfiguracji PHP i serwera przez stronę `phpinfo()`.

Endpoint był możliwy do odkrycia z komentarza HTML i był dostępny bez uwierzytelnienia oraz autoryzacji. Strona ujawniała informacje techniczne, w tym wrażliwy sekret aplikacyjny.

## Dotknięty obszar

```text
GET /cgi-bin/phpinfo.php
```

Wskazówka discovery znaleziona w page source:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

## Mapowanie OWASP

```text
OWASP Top 10 2025: A02 - Security Misconfiguration
```

Powiązany typ słabości:

```text
Information Disclosure
Exposed Debug Functionality
Unsafe Production Configuration
Secret Exposure
```

## Ryzyko / wpływ

Wystawiony debug endpoint ujawniał szczegóły konfiguracji aplikacji i środowiska.

Przykładowe ujawnione informacje:

- wersja PHP,
- szczegóły systemu operacyjnego,
- Server API,
- ścieżki do konfiguracji PHP,
- załadowane pliki konfiguracyjne,
- moduły i rozszerzenia PHP,
- ustawienia PHP,
- wartości środowiskowe/konfiguracyjne,
- wrażliwy sekret aplikacyjny.

Wrażliwa wartość została celowo zredagowana:

```text
<redacted-secret-key>
```

Takie informacje ułatwiają reconnaissance i bardziej ukierunkowane testowanie. Jeśli ujawniony sekret jest używany do podpisywania cookies, sesji, tokenów CSRF, JWT, reset tokenów lub innych zaufanych danych, wpływ może być wysoki.

Dokładny wpływ zależy od aplikacji, ale ujawniony sekret należy traktować jako skompromitowany i obrócić.

## Root cause

Funkcjonalność developerska/diagnostyczna została wdrożona albo pozostawiona w publicznym środowisku.

Debug link był ukryty w komentarzu HTML, ale backendowy endpoint nadal był publicznie osiągalny.

Aplikacja polegała na obscurity zamiast usunąć lub właściwie zabezpieczyć stronę diagnostyczną.

## Dowody

Publiczna strona zawierała komentarz HTML:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

Użytkownik mógł następnie wejść bezpośrednio na debug endpoint:

```http
GET /cgi-bin/phpinfo.php
```

Endpoint zwracał stronę `phpinfo()` zawierającą szczegóły konfiguracji PHP/serwera i wrażliwy `SECRET_KEY`.

Wartości wrażliwe są w raporcie zredagowane.

## Dlaczego to jest Security Misconfiguration

To jest Security Misconfiguration, bo funkcjonalność diagnostyczna przeznaczona dla developmentu lub troubleshooting była publicznie dostępna.

Głównym problemem nie jest XSS, SQL Injection ani Broken Access Control. Głównym problemem jest niebezpieczna konfiguracja produkcyjna i nadmierne ujawnienie informacji.

## Oczekiwane bezpieczne zachowanie

W środowisku produkcyjnym ten endpoint nie powinien być publicznie dostępny.

Preferowane zachowanie:

```http
HTTP/1.1 404 Not Found
```

bo debug file nie powinien być wdrożony.

Jeśli diagnostyka jest celowo wymagana, powinna być ograniczona do zaufanego internal/admin access. Wtedy użytkownik bez uprawnień powinien otrzymać:

```http
HTTP/1.1 403 Forbidden
```

Odpowiedź nie może ujawniać:

- `phpinfo()`,
- wersji PHP,
- wewnętrznych ścieżek,
- zmiennych środowiskowych,
- wartości konfiguracyjnych,
- sekretów,
- debug output.

## Remediacja

Rekomendowane działania:

1. Usuń `phpinfo.php` i inne debug files z produkcyjnych deploymentów.
2. Zablokuj publiczny dostęp do znanych ścieżek debug na poziomie web servera lub reverse proxy.
3. Usuń debug references z HTML comments, templates, JavaScript bundles i publicznych assetów.
4. Obróć ujawniony `SECRET_KEY`, bo powinien być traktowany jako skompromitowany.
5. Sprawdź logi aplikacji, artifacty deploymentu i historię repozytorium pod kątem dodatkowego ujawnienia sekretów.
6. Dodaj secret scanning do CI/CD.
7. Dodaj deployment checks blokujące znane debug files w production artifacts.
8. Jeśli diagnostyka jest potrzebna, trzymaj ją wyłącznie w zaufanych środowiskach.
9. Zreviewuj separację środowisk development, staging i production.
10. Udokumentuj wymagania production hardening.

## Pomysły na testy regresji

Dodaj testy lub deployment checks potwierdzające, że:

```text
GET /cgi-bin/phpinfo.php zwraca 404 albo 403 w środowisku production-like.
Odpowiedź nie zawiera "phpinfo".
Odpowiedź nie zawiera "PHP Version".
Odpowiedź nie zawiera "SECRET_KEY".
Odpowiedź nie zawiera zmiennych środowiskowych.
Production build artifact nie zawiera phpinfo.php.
Publiczny HTML source nie zawiera referencji do debug endpointów.
Secret scanning przechodzi przed deploymentem.
Wcześniej ujawnione sekrety zostały obrócone.
```

## Takeaway developerski

Ukrycie debug linku w komentarzu HTML nie jest kontrolą bezpieczeństwa.

Wszystko wysłane do przeglądarki należy traktować jako widoczne dla użytkownika. Prawdziwa poprawka to usunięcie albo ochrona backendowego endpointu oraz upewnienie się, że funkcjonalność debugowa nie jest wystawiona publicznie na produkcji.

Frontend może wspierać bezpieczeństwo przez unikanie wycieków w HTML, JavaScript, komentarzach i source maps, ale produkcyjne bezpieczeństwo musi być egzekwowane przez backend, konfigurację serwera, deployment controls i secret management.
