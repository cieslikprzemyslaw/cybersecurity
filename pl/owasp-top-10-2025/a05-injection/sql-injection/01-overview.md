# SQL Injection - Overview

## Definicja

SQL Injection to podatność, w której dane kontrolowane przez użytkownika trafiają do zapytania SQL i zmieniają logikę wykonywaną przez bazę danych.

Podatny wzorzec wygląda tak:

```php
$book_name = $_GET['book_name'];
$sql = "SELECT * FROM books WHERE book_name = '$book_name'";
```

Problem polega na tym, że `$book_name` trafia bezpośrednio do stringa SQL. Jeśli użytkownik poda składnię SQL, baza może potraktować część inputu jako wykonywalną logikę zapytania.

## Dlaczego to ważne

SQL Injection może prowadzić do:

- obejścia uwierzytelniania,
- ujawnienia danych wrażliwych,
- ekstrakcji nazw użytkowników i haseł,
- modyfikacji danych,
- usunięcia danych,
- możliwej eskalacji uprawnień zależnie od bazy i konfiguracji.

## Interpreter

Właściwym interpreterem jest **baza danych / silnik SQL**, na przykład MySQL, MariaDB, PostgreSQL, MSSQL, Oracle albo inna baza SQL.

PHP, Node.js, Java albo inny kod backendowy może budować i wysyłać zapytanie, ale finalną instrukcję SQL interpretuje silnik SQL.

## Przyczyna źródłowa

Przyczyna źródłowa to zwykle:

```text
Łączenie niezaufanych danych wejściowych z tekstem zapytania SQL.
```

Walidacja może pomóc, ale realna ochrona wynika z parametryzacji zapytań.

## Główna remediacja

Użyj prepared statements / parameterized queries.

Koncepcyjnie:

```text
Struktura SQL: SELECT * FROM books WHERE book_name = ?
Input usera:   przekazany osobno jako wartość
```

To zapobiega zamianie danych użytkownika w składnię SQL.
