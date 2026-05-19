# PHP - Path Traversal / File Inclusion

## Ryzykowne API i funkcje

| Function / feature | Ryzyko |
|---|---|
| `include()` | Może dołączyć lokalne lub zdalne pliki kontrolowane przez użytkownika |
| `require()` | Może dołączyć lokalne lub zdalne pliki kontrolowane przez użytkownika |
| `include_once()` | Jak include, ale tylko raz |
| `require_once()` | Jak require, ale tylko raz |
| `file_get_contents()` | Może odczytać dowolny plik, jeśli path jest kontrolowany przez użytkownika |
| `readfile()` | Może ujawnić zawartość pliku |
| `fopen()` | Może otworzyć ścieżki kontrolowane przez użytkownika |
| `php://filter` | Może ujawnić kod źródłowy albo transformować streamy |
| `data://` | Może dostarczyć inline data jako stream |
| `allow_url_include` | Może umożliwić RFI |

## Niebezpieczny wzorzec

```php
include($_GET["page"]);
```

Użytkownik kontroluje, jaki plik PHP dołącza.

## Bezpieczniejszy wzorzec

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

## Notatka dla mnie

W PHP include jest szczególnie wrażliwy, bo dostęp do pliku może zmienić się w wykonanie kodu.
