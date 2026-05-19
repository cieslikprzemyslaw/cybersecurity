# Fixes / Review Cheatsheet

## Główna zasada

Input użytkownika nie powinien bezpośrednio decydować, jaki plik po stronie serwera zostanie odczytany albo dołączony.

## Rzeczy do sprawdzenia

| Check | Po co |
|---|---|
| Używanie file IDs zamiast filenames | Zapobiega bezpośredniej manipulacji ścieżką |
| Allowlist mappings | Zapobiega arbitralnemu dostępowi do plików |
| Resolve canonical path | Ujawnia finalną ścieżkę po normalizacji |
| Sprawdzenie, czy path zostaje w base directory | Zapobiega directory breakout |
| Wyłączenie remote includes | Zapobiega RFI |
| Wyłączenie niepotrzebnych wrappers | Ogranicza wrapper-based abuse |
| Wyłączenie verbose errors na produkcji | Zapobiega path disclosure |
| Least privilege | Ogranicza impact file read bugs |
| Aktualny runtime/framework | Usuwa znane historyczne bypassy |
| Regression tests | Zapobiegają powrotowi błędu |

## Zły fix

```js
filename.replace("../", "")
```

To słabe, bo traversal może być nested, encoded, double encoded albo odtworzony po filtrowaniu.

## Bezpieczniejszy wzorzec

1. Nie akceptuj surowych ścieżek od użytkownika.
2. Używaj ID albo allowlisted keys.
3. Mapuj wybór użytkownika na znane pliki po stronie serwera.
4. Resolve final path.
5. Sprawdź, czy final path zostaje wewnątrz allowed base directory.
6. Egzekwuj authorization.
7. Zwracaj generyczne błędy.
8. Dodaj regression tests dla znanych bypassów.

## Przykład allowlist / mapping

Zamiast:

```php
include($_GET["page"]);
```

Użyj:

```php
$allowed = [
    "en" => "languages/EN.php",
    "ar" => "languages/AR.php"
];

$lang = $_GET["lang"] ?? "en";

if (array_key_exists($lang, $allowed)) {
    include($allowed[$lang]);
}
```

Użytkownik kontroluje bezpieczny klucz, a nie faktyczną ścieżkę filesystemu.

## Walidacja canonical path

Aplikacja powinna sprawdzać finalną resolved path.

Przykład koncepcyjny:

```txt
base directory: /var/www/images/
requested path: /var/www/images/../../../etc/passwd
resolved path: /etc/passwd
```

Request powinien zostać zablokowany, bo resolved path nie znajduje się już w `/var/www/images/`.

## Pomysły na regression tests

- `../../../etc/passwd`
- `/etc/passwd`
- `....//....//etc/passwd`
- `%2e%2e%2f`
- `..%252f..%252fetc%252fpasswd`
- `/var/www/images/../../../etc/passwd`
