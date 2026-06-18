# Ćwiczenie 04: Timeout, Retry and Unknown Outcome

## Scenariusz

Frontend wysyła:

```http
PATCH /api/assessments/asm_123/status

{
  "status": "completed"
}
```

Normalny flow:

```text
1. Użytkownik klika Complete Assessment.
2. Backend sprawdza auth/authz.
3. Backend sprawdza state transition.
4. Backend zapisuje status = completed.
5. Backend tworzy activity log entry.
6. Backend zwraca 200 OK.
7. Frontend pokazuje completed.
```

Scenariusz awarii:

```text
Backend kończy operację, ale frontend dostaje timeout, zanim otrzyma response.
```

Frontend widzi tylko:

```text
Network request failed / timeout
```

## Co poprawiłem

Na początku myślałem, że frontend może założyć, że assessment nie został completed, bo response nie dotarł.

Poprawiony model:

```text
Frontend może powiedzieć: sukces nie został potwierdzony.
Frontend nie może stwierdzić jako faktu: backend nic nie zrobił.
```

Timeout oznacza unknown outcome.

Serwer mógł:

- w ogóle nie dostać requestu,
- dostać tylko część requestu,
- dostać request, ale paść przed zmianą stanu,
- zmienić stan, ale paść przed response,
- zakończyć sukcesem, ale response został zgubiony.

## Lekcja A10

Timeout nie jest dowodem porażki. Blind retry może zdublować side effects.

Możliwe zdublowane side effects:

- duplicate activity logs,
- repeated emails or notifications,
- duplicate file uploads,
- repeated payment attempts,
- repeated status-change side effects.

## Bezpieczne oczekiwane zachowanie

Frontend powinien:

- nie pokazywać confirmed success bez potwierdzenia,
- nie zakładać, że backend nic nie zrobił,
- pokazać `unable to confirm` albo kontrolowany error state,
- odświeżyć stan backendu tam, gdzie to możliwe,
- unikać nieskończonych automatic retry loops,
- pozwalać na retry tylko gdy jest bezpieczny albo po sprawdzeniu statusu.

Backend powinien:

- bezpiecznie obsługiwać powtórzone requesty,
- unikać duplikowania side effects, gdy final state już został osiągnięty,
- zwrócić current state, gdy powtórzona operacja jest już complete,
- zachować security invariant.

## Testy regresji

Przydatne testy:

- zasymuluj frontend timeout po backend status update,
- sprawdź, że UI nie pokazuje confirmed success bez potwierdzenia,
- sprawdź, że refresh assessment pokazuje realny backend state,
- retry tej samej zmiany statusu nie duplikuje completion activity log,
- repeated requests zwracają safe current state albo controlled rejection,
- timeout nie zostawia optimistic UI permanentnie pokazującego unconfirmed sensitive action.

## Frontend takeaway

Poprawny stan UI po timeout to nie `success` i niekoniecznie `nothing happened`. To `unknown / unable to confirm` do czasu sprawdzenia stanu backendu.
