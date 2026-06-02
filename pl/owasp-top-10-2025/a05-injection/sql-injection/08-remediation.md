# SQL Injection Remediation

## Główna poprawka

Główna poprawka dla SQL Injection to:

```text
Prepared statements / parameterized queries
```

To oddziela strukturę SQL od wartości kontrolowanych przez użytkownika.

## Podatny koncept

```text
tekst zapytania SQL + input użytkownika = niebezpieczne
```

Przykładowa podatna idea:

```php
$sql = "SELECT * FROM books WHERE book_name = '$book_name'";
```

Jeśli `book_name` zawiera składnię SQL, może zmienić zapytanie.

## Bezpieczny koncept

```text
Struktura SQL jest stała.
Input użytkownika jest przekazywany osobno jako wartość parametru.
```

Przykładowo:

```text
SELECT * FROM books WHERE book_name = ?
wartość parametru = nazwa książki podana przez użytkownika
```

Baza traktuje input jako dane, a nie wykonywalną składnię SQL.

## Dodatkowe warstwy

Prepared statements powinny być wspierane przez inne kontrole:

- allowlisty wartości, gdzie to możliwe,
- walidację typu i formatu,
- bezpieczne wzorce ORM,
- unikanie unsafe raw queries,
- brak szczegółowych błędów bazy w produkcji,
- konta bazy danych z least privilege,
- monitoring podejrzanego zachowania zapytań,
- testy regresji.

## GET vs POST

Zmiana requestu z GET na POST nie naprawia SQL Injection.

GET vs POST zmienia miejsce przesyłania inputu, a nie to, czy jest bezpieczny.

```text
Granicą bezpieczeństwa jest sposób, w jaki backend buduje i wykonuje zapytanie SQL.
```

POST albo JSON body mogą porządkować projekt API, ale same w sobie nie zapobiegają SQLi.

## Wniosek dla developera

Nie ufaj inputowi tylko dlatego, że pochodzi z:

- URL wygenerowanego przez aplikację,
- dropdowna,
- cookie,
- hidden field,
- wartości kontrolowanej przez frontend,
- pola zarządzanego w CMS.

Backend musi obsługiwać każdy input bezpiecznie.
