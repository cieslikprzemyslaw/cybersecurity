# Timeouts, Resources and Race Conditions

## Timeout means unknown outcome

Timeout oznacza, że klient nie dostał odpowiedzi na czas. Nie dowodzi, że backend nie dostał requestu, nie przetworzył go, wykonał rollback albo że retry jest bezpieczny.

Bezpieczniejsze sformułowanie po stronie frontendu:

```text
The operation could not be confirmed.
```

nie:

```text
The operation definitely failed and nothing happened.
```

## Dependency failure

Sprawdzam, czy fallback nie jest mniej bezpieczny niż normal path:

- cannot verify authorization -> allow request,
- cannot scan uploaded file -> accept as safe,
- cannot validate token -> treat as logged in,
- cannot record audit event -> perform sensitive action with no accountability.

## Resource exhaustion

Exceptional conditions mogą prowadzić do DoS, gdy resources nie są limitowane albo zwalniane: file handles, database connections, locks, temporary files, repeated retries, huge bodies, expensive error paths.

Kontrole: request-size limits, rate limits, timeouts, quotas, concurrency limits, bounded queues, cleanup in `finally`, monitoring repeated failures.

## Race condition basics

```text
two requests
    -> same mutable state
    -> check before update
    -> both pass the check
    -> invalid outcome
```

Przykład: dwa requesty czytają `status=in-progress`, oba ustawiają `completed` i oba tworzą activity event. Final status wygląda OK, ale duplicate side effects istnieją.

## Disabled button is not enough

Disabled button pomaga w UX, ale nie zatrzymuje Burpa, ZAP, Postmana, DevTools, skryptów, retry przeglądarki, dwóch kart albo duplikatów wynikających z wolnej sieci. Backend musi zachować invariant.
