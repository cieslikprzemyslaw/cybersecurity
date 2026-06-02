# SQL Injection

SQL Injection występuje wtedy, gdy niezaufane dane wejściowe zostają umieszczone w zapytaniu SQL w sposób pozwalający zmienić logikę wykonywaną przez silnik bazy danych.

## Model mentalny

```text
dane kontrolowane przez użytkownika
  -> string zapytania SQL
  -> baza danych / interpreter SQL
  -> zmienione zachowanie zapytania
```

## Pliki

- [01-overview.md](01-overview.md) - definicja, impact, interpreter, przyczyna źródłowa i główna remediacja.
- [02-types-of-sqli.md](02-types-of-sqli.md) - główne typy SQLi i wzorce dowodów.
- [03-in-band-sqli.md](03-in-band-sqli.md) - error-based i UNION-based SQLi.
- [04-union-based-sqli.md](04-union-based-sqli.md) - notatki o UNION i flow dowodów z laba.
- [05-blind-sqli.md](05-blind-sqli.md) - boolean-based blind SQLi i myślenie o oracle.
- [06-out-of-band-sqli.md](06-out-of-band-sqli.md) - koncepcje OOB i ograniczenia środowiskowe.
- [07-filter-bypass-and-encoding.md](07-filter-bypass-and-encoding.md) - encoding i obchodzenie filtrów.
- [08-remediation.md](08-remediation.md) - model remediacji dla developera.
- [cheat-sheet.md](cheat-sheet.md) - praktyczne przypomnienia do testowania i review.
- [regression-tests.md](regression-tests.md) - testy regresji specyficzne dla SQLi.
- [learning-summary.md](learning-summary.md) - porównanie ukończonych labów i wnioski.

## Ukończone laby

- [UNION-based SQL Injection - retrieving data from other tables](labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](labs/02-blind-conditional-responses.md)

## Najważniejszy wniosek

Główna poprawka dla SQL Injection to nie silniejsze blacklisty ani zmiana GET na POST. Główna poprawka to prepared statements / parameterized queries, które rozdzielają strukturę SQL od danych użytkownika.
