# NoSQL Injection

NoSQL Injection występuje wtedy, gdy niezaufany input zostaje użyty do zbudowania zapytania NoSQL w sposób, który zmienia zamierzoną logikę query.

## Model mentalny

```text
input kontrolowany przez użytkownika
  -> wartość prymitywna, zagnieżdżony obiekt, operator query albo wyrażenie
  -> baza NoSQL / query engine
  -> zmienione zachowanie zapytania
```

Kluczowe pytanie:

```text
Czy aplikacja utrzymała input jako dane,
czy input stał się częścią query?
```

## Główne formy w tym module

### Operator Injection

Atakujący powoduje, że input, który powinien być prostą wartością, staje się zagnieżdżonym operatorem albo obiektem query.

Przykłady:

- `$ne`
- `$nin`
- `$gt`
- `$regex`

### Syntax Injection

Atakujący wychodzi z wyrażenia query i dodaje nową składnię. W MongoDB może to wystąpić, gdy aplikacja buduje custom JavaScript-style expressions, na przykład przez `$where`.

To zwykle jest mniej powszechne niż Operator Injection, bo wymaga niebezpiecznej custom query construction.

## Pliki

- [01-overview.md](01-overview.md) - definicja NoSQL Injection, model mentalny, impact i przyczyny źródłowe.
- [02-mongodb-data-model-and-query-filters.md](02-mongodb-data-model-and-query-filters.md) - dokumenty, kolekcje, filtry i operatory MongoDB.
- [03-operator-injection.md](03-operator-injection.md) - manipulacja obiektami/operatorami i auth bypass.
- [04-syntax-injection.md](04-syntax-injection.md) - custom JavaScript expressions, test apostrofem i warunki always-true.
- [05-blind-nosql-injection-and-data-extraction.md](05-blind-nosql-injection-and-data-extraction.md) - boolean oracles i ekstrakcja znak po znaku.
- [06-remediation.md](06-remediation.md) - model remediacji dla developera.
- [cheat-sheet.md](cheat-sheet.md) - praktyczne przypomnienia do testów i code review.
- [regression-tests.md](regression-tests.md) - pomysły na testy regresji specyficzne dla NoSQL.
- [learning-summary.md](learning-summary.md) - podsumowanie procesu nauki i wnioski.

## Ukończone laby

- [Detecting NoSQL injection](labs/01-detecting-nosql-injection.md)
- [Exploiting NoSQL injection to extract data](labs/02-extracting-data-with-a-boolean-oracle.md)
- [Porównanie labów](labs/summary.md)

## Źródła wspierające

- TryHackMe: NoSQL Injection
- PortSwigger Web Security Academy: NoSQL injection
- MongoDB Manual: [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/)

## Najważniejszy wniosek

Główna obrona to nie blacklistowanie `$` albo apostrofów. Aplikacja musi egzekwować oczekiwane typy i schematy, nie wpuszczać inputu klienta do struktury query, unikać budowania custom JavaScript query i weryfikować hasła poza zapytaniem do bazy przez bezpieczne porównanie z hashem.
