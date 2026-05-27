# A02 Learning notes: Security Misconfiguration

Te notatki zachowują dłuższe refleksje z labów A02. Krótszy zapis praktyki znajduje się w [02-labs-or-practice.md](02-labs-or-practice.md).

## Czego się nauczyłem

### Security Misconfiguration często wygląda jak Information Disclosure

A02 nie zawsze oznacza skomplikowany exploit. Często chodzi o coś, co nie powinno być widoczne, ale jest widoczne przez niebezpieczną konfigurację.

Przykłady z tych labów:

- verbose error output,
- traceback disclosure,
- publiczny debug endpoint,
- publiczne `phpinfo()`,
- wyciek konfiguracji lub środowiska,
- ujawniony sekret aplikacyjny.

### Invalid input jest dobry do testowania obsługi błędów

Jeśli aplikacja mówi:

```text
User ID must be numeric
```

warto sprawdzić, co stanie się po podaniu wartości nienumerycznej.

Celem nie zawsze jest SQL Injection albo IDOR. Czasem najważniejsze jest to, jak aplikacja się wywraca.

Przykładowe wartości testowe:

```text
admin
abc
test
-1
0
1.5
999999999999999999999
'
%27
```

### HTML comments mogą ujawniać recon clues

HTML comments mogą ujawniać:

- ukryte endpointy,
- starą funkcjonalność,
- debug tools,
- wewnętrzne notatki,
- tymczasowe linki,
- development-only routes.

Komentarz nie zabezpiecza niczego. Jeśli został wysłany do przeglądarki, jest widoczny dla użytkownika.

### Debug pages są niebezpieczne publicznie

Debug pages takie jak `phpinfo()` mogą ujawnić bardzo dużo informacji technicznych.

Nawet jeśli część ustawień wygląda bezpiecznie, sama strona może nadal ujawniać środowisko, ścieżki, moduły i sekrety.

### Sekrety nie mogą trafiać do błędów ani debug outputu

Jeśli sekret został ujawniony, poprawka powinna obejmować:

- usunięcie ekspozycji,
- rotację sekretu,
- sprawdzenie, gdzie sekret był używany,
- review logów i deployment artifacts,
- dodanie secret scanning.

## Co mnie zmyliło lub wymagało doprecyzowania

### `400` vs `404` vs `403`

Dla invalid input do istniejącego endpointu zwykle pasuje `400 Bad Request`.

Przykład:

```http
GET /api/user/admin
```

Endpoint istnieje, ale input ma niepoprawny format.

Dla debug page, która nie powinna istnieć publicznie, lepsze jest zwykle `404 Not Found`.

Przykład:

```http
GET /cgi-bin/phpinfo.php
```

Jeśli endpoint diagnostyczny istnieje celowo, ale tylko dla uprawnionych użytkowników, wtedy dla użytkownika bez uprawnień pasuje `403 Forbidden`.

### Komentarz nie był podatnością sam w sobie

HTML comment był wskazówką discovery.

Prawdziwą podatnością był publiczny debug endpoint oraz ujawniona konfiguracja/sekret.

### Usunięcie komentarza nie wystarcza

Usunięcie HTML comment zmniejsza discoverability, ale nie naprawia problemu, jeśli `/cgi-bin/phpinfo.php` nadal istnieje.

Endpoint trzeba usunąć, zablokować lub prawidłowo zabezpieczyć.

### Zmiana URL-a nie jest właściwą poprawką

Zmiana `/cgi-bin/phpinfo.php` na inną ukrytą ścieżkę to security by obscurity.

Lepsza poprawka to usunięcie debug file z produkcji albo prawidłowe ograniczenie dostępu.

## Real-world review angle

W prawdziwej aplikacji szukałbym tego patternu w:

- HTML comments,
- JavaScript bundles,
- source maps,
- `robots.txt`,
- `sitemap.xml`,
- starych test routes,
- admin/debug links,
- `/debug`,
- `/config`,
- `/status`,
- `/health`,
- `/server-status`,
- `/actuator`,
- `/phpinfo.php`,
- `/cgi-bin/phpinfo.php`,
- `.env`,
- `.bak`,
- `.old`,
- log files,
- verbose API errors,
- stack traces,
- response headers,
- deployment artifacts.

Sprawdziłbym też, czy debug tools albo diagnostic pages nie zostały wdrożone do produkcji.

## Podsumowanie

Dwa praktyczne patterny A02 z tych labów:

1. invalid input powodujący verbose debug/traceback disclosure,
2. publiczny debug endpoint ujawniający `phpinfo()` i sekret aplikacyjny.

Główna lekcja:

> Systemy produkcyjne powinny failować bezpiecznie i nie ujawniać użytkownikom debug information, stack traces, diagnostic pages, sekretów, szczegółów środowiska ani konfiguracji wewnętrznej.
