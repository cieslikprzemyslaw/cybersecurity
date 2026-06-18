# A10 Labs and Practice

A10 nie miało dedykowanego laba TryHackMe w roomie, który sprawdziłem na początku, dlatego te notatki dokumentują praktyczne ćwiczenia review wykonane podczas sprintu.

## Ukończona praktyka

1. [API error-handling review](01-api-error-handling-review.md)
2. [Transaction and rollback review](02-transaction-and-rollback-review.md)
3. [Missing and malformed parameters](03-missing-and-malformed-parameters.md)
4. [Timeout, retry and unknown outcome](04-timeout-retry-and-unknown-outcome.md)
5. [Duplicate submit and race-condition basics](05-duplicate-submit-and-race-basics.md)

## Cel

Celem nie było zostanie seniorem od backendu, SRE ani distributed systems. Celem było przećwiczenie pytań AppSec review z perspektywy Frontend Engineera:

```text
Co się zepsuło?
Jaki stan już się zmienił?
Czy system działa fail-open czy fail-closed?
Czy operacja została wykonana częściowo?
Czy retry może zdublować side effects?
Czy frontend pokazuje stan potwierdzony, czy tylko niepotwierdzony?
Jakie zachowanie backendu musi zachować security invariant?
```

## Wynik końcowy

PASS — potrafię rozpoznać niebezpieczną obsługę exceptional conditions i zaproponować praktyczne wymagania oraz testy regresji na poziomie Frontend Engineer -> AppSec.
