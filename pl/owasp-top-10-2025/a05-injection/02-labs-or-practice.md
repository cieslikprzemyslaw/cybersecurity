# A05: Injection - Laby i praktyka

Ta kategoria zawiera obecnie praktykę dla SQL Injection i NoSQL Injection.

## Ukończone laby SQL Injection

- [UNION-based SQL Injection - retrieving data from other tables](sql-injection/labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](sql-injection/labs/02-blind-conditional-responses.md)

## Ukończona praktyka NoSQL Injection

### TryHackMe

- NoSQL Injection room
- podstawy dokumentów i kolekcji MongoDB,
- filtry zapytań MongoDB i zagnieżdżone operatory,
- obejście uwierzytelniania przez Operator Injection,
- ekstrakcja true/false przy użyciu `$regex`,
- świadomość Syntax Injection w custom JavaScript-style queries.

### PortSwigger Web Security Academy

- [Detecting NoSQL injection](nosql-injection/labs/01-detecting-nosql-injection.md)
- [Exploiting NoSQL injection to extract data](nosql-injection/labs/02-extracting-data-with-a-boolean-oracle.md)
- [Porównanie labów NoSQL](nosql-injection/labs/summary.md)

## Pokrycie praktyczne

Ukończona praktyka obejmuje:

- identyfikowanie danych kontrolowanych przez użytkownika, które trafiają do zapytania bazy,
- odróżnianie prostej wartości od zagnieżdżonego obiektu, operatora lub wyrażenia,
- rozpoznawanie dowodów injection w odpowiedziach HTTP,
- używanie baseline'u przed interpretacją błędów lub zmian odpowiedzi,
- porównywanie warunków true i false,
- używanie zachowania odpowiedzi jako boolean oracle,
- ustalanie długości sekretu,
- ekstrakcję znaków sekretu po jednej pozycji,
- używanie Burp Intruder dopiero po ręcznym potwierdzeniu oracle,
- przekładanie dowodów technicznych na remediację i testy regresji.

## Aktualne ograniczenia

Ten katalog A05 nie zawiera jeszcze ukończonych modułów praktycznych dla:

- OS Command Injection,
- Server-Side Template Injection,
- AI Prompt Injection,
- mapowania XSS pod A05.

Te tematy powinny zostać dodane dopiero po ukończeniu odpowiedniej nauki i praktyki.
