# NoSQL Injection - Overview

## Czym jest NoSQL Injection?

NoSQL Injection występuje wtedy, gdy niezaufany input zmienia znaczenie zapytania wykonywanego przez bazę NoSQL albo query engine.

NoSQL to szerokie pojęcie. Różne bazy NoSQL używają różnych modeli przechowywania danych, języków zapytań, API i operatorów. Składnia payloadu zależy więc od bazy, języka, frameworka, drivera i parsera requestu.

Ta sekcja skupia się głównie na przykładach w stylu MongoDB.

## Model danych MongoDB

MongoDB przechowuje dane w dokumentach, a nie relacyjnych wierszach.

Uproszczony dokument może wyglądać tak:

```json
{
  "_id": "example-id",
  "username": "lphillips",
  "first_name": "Logan",
  "last_name": "Phillips",
  "age": "65",
  "email": "lphillips@example.com"
}
```

Powiązane dokumenty są grupowane w kolekcje. Kolekcje są grupowane w bazy danych.

Orientacyjne porównanie:

```text
Relacyjna baza danych   MongoDB
---------------------   -------
wiersz / rekord         dokument
tabela                  kolekcja
baza danych             baza danych
warunek WHERE           filter query
```

To porównanie pomaga orientacyjnie, ale dokumenty MongoDB i relacyjne wiersze nie są identycznymi strukturami danych.

## Główne formy NoSQL Injection

### Operator Injection

Operator Injection występuje wtedy, gdy input kontrolowany przez atakującego jest interpretowany jako operator query albo zagnieżdżony obiekt, a nie prosta wartość.

Zmiana mentalna:

```text
Oczekiwane:
username równa się podanemu stringowi

Niebezpieczny rezultat:
username nie jest równy podanemu stringowi
```

### Syntax Injection

Syntax Injection występuje wtedy, gdy input wychodzi z wyrażenia query i dodaje nową składnię.

Koncepcyjnie przypomina SQL Injection, ale składnia zależy od konkretnego mechanizmu NoSQL. W MongoDB ryzykownym przykładem jest doklejanie inputu użytkownika do custom JavaScript expression przez `$where`.

Syntax Injection zwykle jest mniej powszechne niż Operator Injection, bo wymaga niebezpiecznego budowania custom expression.

## SQL Injection vs NoSQL Injection

### SQL Injection

```text
input użytkownika -> string SQL -> parser SQL -> zmienione query SQL
```

Typowy fokus:

- apostrofy,
- komentarze,
- UNION,
- subqueries,
- funkcje specyficzne dla SQL.

### NoSQL Injection

```text
input użytkownika -> wartość/obiekt/operator/wyrażenie -> NoSQL query engine -> zmienione query
```

Typowy fokus:

- zagnieżdżony JSON albo tablice asocjacyjne,
- zachowanie parsera,
- input w kształcie operatora,
- dynamiczne obiekty,
- custom JavaScript expressions,
- różnice odpowiedzi true/false.

NoSQL Injection to nie SQL Injection z innymi słowami kluczowymi. Struktura danych w requestcie często ma takie samo znaczenie jak widoczny tekst.

## Typowy attack surface

- formularze logowania,
- endpointy user lookup,
- API wyszukiwania i filtrowania,
- filtry kategorii produktów,
- endpointy profilu,
- JSON APIs,
- URL-encoded form bodies,
- query-string parameters,
- background fetch/XHR requests,
- filtry resolverów GraphQL lub API,
- administracyjne funkcje wyszukiwania.

## Możliwy impact

- auth bypass,
- nieautoryzowany dostęp do rekordów,
- user enumeration,
- ujawnienie wrażliwych pól,
- blind extraction sekretów,
- przejęcie konta administratora,
- obejście filtrowania,
- ujawnienie unreleased albo hidden data,
- verbose error i technology disclosure.

## Przyczyny źródłowe

- akceptowanie obiektów tam, gdzie oczekiwane są stringi,
- brak walidacji schematu,
- brak egzekwowania typów,
- przekazywanie obiektów requestu bezpośrednio do filtrów bazy,
- pozwalanie klientowi na wybór operatorów query,
- niebezpieczne dynamic query construction,
- doklejanie inputu do `$where` albo custom JavaScript,
- odpytywanie plaintext passwords,
- zaufanie do wartości wygenerowanych przez frontend,
- różne odpowiedzi tworzące stabilny oracle.

## Pytania AppSec

1. Jaki input kontroluję?
2. Jaki typ danych otrzymuje backend?
3. Jaki parser przetwarza request?
4. Czy input jest wartością, czy strukturą query?
5. Jaki database/query system go otrzymuje?
6. Jaka odpowiedź dowodzi, że query się zmieniło?
7. Co jest faktem, a co tylko założeniem?
8. Jaka bezpieczna implementacja utrzymałaby strukturę query po stronie serwera?
