# A10: Mishandling of Exceptional Conditions

Ten katalog zawiera moje praktyczne notatki do OWASP Top 10 2025 — A10: Mishandling of Exceptional Conditions.

## Status

PASS — Frontend Developer przechodzący do AppSec

Ukończona praca:

- przegląd teorii OWASP A10:2025,
- przegląd OWASP Error Handling Cheat Sheet,
- review obsługi błędów w API,
- review partial state i rollbacku,
- review brakujących i nieprawidłowych parametrów,
- review timeoutów, retry i unknown outcome,
- podstawy duplicate submit i race condition,
- porównanie A10 z A01 Broken Access Control, A06 Insecure Design i A09 Security Logging and Alerting Failures.

Ten status nie oznacza eksperckiej wiedzy z distributed systems, database transactions, SRE ani zaawansowanej współbieżności. Oznacza, że potrafię rozpoznać niebezpieczne zachowanie po awarii, odróżnić fail-open od fail-closed, przejrzeć podstawową obsługę błędów, wskazać partial state oraz zaproponować praktyczne wymagania bezpieczeństwa i testy regresji.

## Główny model mentalny

```text
unexpected condition
    -> detection
    -> safe handling
    -> rollback / cleanup / recovery
    -> generic external response
    -> internal logging and alerting
```

## Najważniejsze pytania

1. Co miało się normalnie wydarzyć?
2. Jaka sytuacja wyjątkowa wystąpiła?
3. Jaki stan zdążył się już zmienić?
4. Czy system zadziałał fail-open czy fail-closed?
5. Czy operacja została wykonana częściowo?
6. Czy wymagany był rollback, cleanup albo recovery?
7. Czy odpowiedź ujawniła wewnętrzne szczegóły?
8. Czy retry może zdublować side effect?
9. Czy ten sam warunek może wyczerpać zasoby albo zostać nadużyty?
10. Jaki test regresji dowodzi powrotu do bezpiecznego stanu?

## Perspektywa frontendu

Kontrole frontendowe poprawiają UX, ale nie są autorytatywnymi kontrolami bezpieczeństwa.

- React error boundary poprawia odporność UI, nie odzyskanie stanu po stronie backendu.
- Generyczny błąd na froncie nie dowodzi rollbacku.
- Odpowiedź `500` nie dowodzi, że nic się nie zmieniło.
- Timeout oznacza brak potwierdzenia; serwer mógł już przetworzyć request.
- Optimistic UI nie może zostawić wrażliwej akcji jako sukcesu bez potwierdzenia backendu.
- Disabled button nie zatrzymuje Burpa, ZAP, Postmana, DevTools, skryptów, retry ani dwóch kart.
- Walidacja backendowa, authentication, authorization, state transitions i transaction behaviour pozostają source of truth.

## Zacznij tutaj

- [Overview](01-overview.md)
- [Fail-open and fail-closed](02-fail-open-and-fail-closed.md)
- [Error handling and information disclosure](03-error-handling-and-information-disclosure.md)
- [Transactions, retries and idempotency](04-transactions-retries-and-idempotency.md)
- [Timeouts, resources and race conditions](05-timeouts-resources-and-race-conditions.md)
- [Review checklist](06-checklist.md)
- [Regression tests](07-regression-tests.md)
- [Learning notes](08-learning-notes.md)
- [Labs and practice](labs-and-practice/README.md)
- [Przykładowe znalezisko](security-findings/01-example-finding-insecure-exception-handling.md)

## Evidence z praktyki

Praktyczna część tematu jest opisana w osobnych plikach:

- [API error-handling review](labs-and-practice/01-api-error-handling-review.md)
- [Transaction and rollback review](labs-and-practice/02-transaction-and-rollback-review.md)
- [Missing and malformed parameters](labs-and-practice/03-missing-and-malformed-parameters.md)
- [Timeout, retry and unknown outcome](labs-and-practice/04-timeout-retry-and-unknown-outcome.md)
- [Duplicate submit and race-condition basics](labs-and-practice/05-duplicate-submit-and-race-basics.md)

## Bezpośrednie linki

- OWASP A10:2025 — Mishandling of Exceptional Conditions: https://owasp.org/Top10/2025/A10_2025-Mishandling_of_Exceptional_Conditions/
- OWASP Error Handling Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
