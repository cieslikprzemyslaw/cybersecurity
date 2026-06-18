# A10 Learning Notes

## Początkowe rozumienie

Na początku opisałem A10 jako nieuwzględnienie wyjątków, których nie przewidzieliśmy. To było blisko, ale za wąsko.

Lepszy model:

```text
Problemem nie jest samo to, że błąd wystąpił.
Problemem jest niebezpieczne zachowanie po sytuacji wyjątkowej.
```

Taki warunek może być przewidziany jako możliwy typ awarii, nawet jeśli dokładna runtime cause jest nieznana. Przykłady: dependency timeout, database failure, missing state, malformed data, failed cleanup, concurrent requests i resource exhaustion.

## Najważniejsze doprecyzowania

### Generic 500 to tylko odpowiedź dla klienta

Generyczna odpowiedź ogranicza information disclosure, ale nie dowodzi rollbacku, cleanupu ani bezpiecznego odzyskania stanu.

Odpowiedź mówi mi tylko, co dostał klient. Nie mówi, co stało się z rekordami w bazie, plikami, emailami, logami, kolejkami, sesjami albo third-party side effects.

### Timeout oznacza unknown outcome

Browser timeout oznacza, że frontend nie dostał potwierdzenia. Backend mógł już odebrać i przetworzyć request.

Bezpieczny UI state:

```text
unable to confirm
```

nie:

```text
nothing happened
```

### Frontend controls to kontrolki UX

Frontend validation, disabled buttons i React error boundaries pomagają użytkownikom, ale nie egzekwują bezpieczeństwa po stronie backendu.

Mogą zmniejszyć przypadkowe błędy, ale nie zatrzymują Burpa, ZAP, Postmana, DevTools, skryptów, wielu kart ani direct API requests.

### Rozróżnienie A01 / A06 / A10

```text
No authorization check exists -> A01 / A06
Authorization check exists but fails open during dependency failure -> A10
```

Missing auth/authz zwykle nie jest `500` i nie jest automatycznie A10. Staje się A10, gdy istniejąca kontrola trafia w sytuację wyjątkową, a aplikacja reaguje niebezpiecznie.

## Ukończona praktyka

### API error handling

Przećwiczyłem przypadki validation, not-found, access i unexpected exception.

Learning:

```text
400 = malformed or invalid request
401 = unauthenticated
403 = authenticated but not allowed
404 = not found or intentionally hidden
500 = unexpected server-side or dependency failure
```

### Partial state

Scenariusz:

```text
create evidence record -> save file -> update assessment count -> failure
```

Learning:

```text
500 nie dowodzi, że nic się nie zmieniło.
Backend nie powinien zostawiać aktywnego niespójnego stanu.
Regression tests powinny potwierdzać rollback albo cleanup.
```

### Missing and malformed parameters

Scenariusz:

```text
PATCH assessment status with {}, "", null, ["completed"], "admin" and "completed"
```

Learning:

```text
Bad input shape powinien zostać odrzucony jako controlled validation failure.
Valid input value nadal wymaga authentication, authorization i allowed state transition checks.
```

### Timeout and retry

Learning:

```text
timeout = unknown outcome
retry może zdublować side effects
frontend powinien odświeżyć stan backendu
backend powinien bezpiecznie obsłużyć repeated requests
```

### Duplicate submit / race basics

Learning:

```text
disabled button nie wystarcza
backend musi zachować invariant
two concurrent complete requests must not create two completion events
```

## Moja praktyczna checklista review

Gdy reviewuję flow, pytam:

1. Jaki jest normal flow?
2. Jaka sytuacja wyjątkowa może wystąpić?
3. Jaki stan mógł już się zmienić?
4. Czy system fail-open czy fail-closed?
5. Czy istnieje partial state?
6. Czy potrzebny jest rollback, cleanup albo recovery?
7. Co powinien zobaczyć użytkownik?
8. Co powinno być zalogowane wewnętrznie?
9. Czy retry może zdublować side effect?
10. Jaki regression test dowodzi bezpiecznego stanu?

## Interview-style explanation

> A10 dotyczy niebezpiecznego zachowania aplikacji, gdy wystąpi sytuacja wyjątkowa. Nie traktowałbym każdego exception jako vulnerability. Sprawdziłbym, jaki warunek wystąpił, jaki stan zdążył się zmienić, czy system fail-open czy fail-closed, czy pozostał partial state, czy retry może zdublować side effects, czy klient zobaczył sensitive internal details i jaki test regresji dowodzi powrotu do known safe state.
