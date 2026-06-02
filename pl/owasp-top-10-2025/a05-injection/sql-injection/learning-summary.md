# SQL Injection Learning Summary

## Ukończone laby

- [UNION-based SQL Injection - retrieving data from other tables](labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](labs/02-blind-conditional-responses.md)

## Lekcja UNION-based SQLi

Lab UNION pokazał, jak podatny parametr `category` może pobrać dane z innej tabeli.

Kluczowe dowody:

```text
apostrof zepsuł zapytanie
komentarz SQL naprawił zapytanie
testy NULL potwierdziły liczbę kolumn
abc/def potwierdziły widoczne kolumny tekstowe
dane z tabeli users pojawiły się w HTML
```

Główna lekcja:

```text
UNION-based SQLi może ujawnić dane bezpośrednio w normalnej odpowiedzi webowej.
```

## Lekcja blind SQLi

Lab blind używał cookie `TrackingId` i markera odpowiedzi, żeby wnioskować o danych bez bezpośredniego ich wyświetlania.

Kluczowe dowody:

```text
Welcome back = TRUE
Brak Welcome back = FALSE
```

Główna lekcja:

```text
Blind SQLi używa zachowania odpowiedzi jako oracle dla pytań tak/nie do bazy.
```

## Porównanie

| Temat | UNION-based SQLi | Blind SQLi |
|---|---|---|
| Dane widoczne bezpośrednio? | Tak | Nie |
| Główny dowód | Dane pojawiają się w odpowiedzi | Marker odpowiedzi się zmienia |
| Przykładowy marker | Wyciągnięte wiersze | `Welcome back` |
| Technika | `UNION SELECT` | Warunki TRUE/FALSE |
| Impact | Ujawnienie danych/przejęcie konta | Ekstrakcja danych/przejęcie konta |

## Ogólny wniosek AppSec

SQL Injection to nie tylko payload. Chodzi o udowodnienie, że input użytkownika zmienia zachowanie zapytania SQL, oraz pokazanie realnego wpływu.

Poprawka developerska to prepared statements / parameterized queries, wspierane przez walidację, allowlisty, bezpieczne użycie ORM, least privilege i testy regresji.
