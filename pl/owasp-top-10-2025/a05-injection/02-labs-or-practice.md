# A05: Injection - Laby i praktyka

Ta kategoria używa obecnie SQL Injection jako praktycznego przykładu A05.

## Ukończone laby SQL Injection

- [UNION-based SQL Injection - retrieving data from other tables](sql-injection/labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](sql-injection/labs/02-blind-conditional-responses.md)

## Pokrycie praktyczne

Ukończona praktyka obejmuje:

- identyfikowanie danych kontrolowanych przez użytkownika, które trafiają do zapytania SQL,
- rozpoznawanie widocznych dowodów SQLi w odpowiedziach HTTP,
- używanie `UNION SELECT` do potwierdzenia ekspozycji danych w legalnym labie,
- używanie różnic w odpowiedzi do rozumowania o blind SQLi,
- przekładanie dowodów z laba na język security findingu,
- definiowanie remediacji i testów regresji.

## Aktualne ograniczenia

Ten katalog A05 nie zawiera jeszcze osobnej praktyki dla NoSQL Injection, OS Command Injection, SSTI, prompt injection ani innych klas injection zależnych od konkretnego interpretera.

Te tematy powinny zostać dodane jako osobne moduły po wykonaniu praktycznych labów lub zadań review.
