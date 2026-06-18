# A10 Practice Summary

## Ukończone ćwiczenia

1. [API error-handling review](01-api-error-handling-review.md)
2. [Transaction and rollback review](02-transaction-and-rollback-review.md)
3. [Missing and malformed parameters](03-missing-and-malformed-parameters.md)
4. [Timeout, retry and unknown outcome](04-timeout-retry-and-unknown-outcome.md)
5. [Duplicate submit and race-condition basics](05-duplicate-submit-and-race-basics.md)

## Status końcowy

PASS — Frontend Developer przechodzący do AppSec.

## Główny model praktyczny

```text
abnormal condition
-> jaki stan już się zmienił?
-> czy aplikacja fail-open czy fail-closed?
-> czy istnieje partial state?
-> czy retry jest bezpieczny?
-> co powinien pokazać frontend?
-> co musi wymusić backend?
-> jaki test regresji dowodzi bezpiecznego stanu?
```

## Co potrafię wyjaśnić

- A10 to niebezpieczne zachowanie podczas sytuacji wyjątkowych, a nie każdy exception.
- Expected validation failures powinny dawać kontrolowane `4xx` responses.
- Unexpected server albo dependency failures mogą dawać kontrolowane `5xx` responses.
- Fail-open oznacza kontynuowanie, gdy kontrola nie może podjąć bezpiecznej decyzji.
- Fail-closed oznacza zachowanie security rule i safe state.
- Generic error responses ograniczają information disclosure, ale nie dowodzą odzyskania bezpiecznego stanu.
- `try/catch` i logging same w sobie nie dowodzą safe handling.
- Timeout oznacza unknown outcome.
- Retry może zdublować side effects.
- Frontend validation i disabled buttons to UX controls, nie backend security controls.
- Missing auth/authz to zwykle A01/A06; unsafe behaviour, gdy istniejąca kontrola pada, może być A10.

## Evidence z ćwiczeń

### 1. API error handling

Przećwiczyłem rozróżnienie przypadków validation, access, not-found i unexpected failure.

Key takeaway:

```text
400 = malformed or invalid request
401 = unauthenticated
403 = authenticated but not allowed
404 = not found or intentionally hidden
500 = unexpected server-side or dependency failure
```

### 2. Transaction and rollback

Przećwiczyłem multi-step evidence upload flow.

Key takeaway:

```text
500 nie dowodzi, że nic się nie zmieniło.
Partial state może pozostać, jeśli jeden krok się udał, a kolejny padł.
```

### 3. Missing and malformed parameters

Przećwiczyłem missing, empty, null, array i unknown enum values.

Key takeaway:

```text
Unexpected input shapes powinny zostać odrzucone jako controlled validation failures, zanim trafią do business logic.
```

### 4. Timeout and retry

Przećwiczyłem frontend timeout po backend status update.

Key takeaway:

```text
timeout = unknown outcome
unknown outcome nie powinien być ślepo ponawiany ani pokazywany jako confirmed success
```

### 5. Duplicate submit / race basics

Przećwiczyłem dwa concurrent `Complete Assessment` requests.

Key takeaway:

```text
disabled button to UX, nie security
backend musi zachować invariant: one completion -> one completion event
```

## Interview-style explanation

> A10 dotyczy niebezpiecznego zachowania aplikacji, gdy wystąpi sytuacja wyjątkowa. Nie traktowałbym każdego exception jako vulnerability. Sprawdziłbym, jaki warunek wystąpił, jaki stan zdążył się zmienić, czy system fail-open czy fail-closed, czy pozostał partial state, czy retry może zdublować side effects, czy klient zobaczył sensitive internal details i jaki test regresji dowodzi powrotu do known safe state.
