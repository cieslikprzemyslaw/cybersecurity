# A10 — Overview

## Definicja

A10 występuje wtedy, gdy aplikacja nie potrafi bezpiecznie zapobiegać, wykrywać, obsługiwać, odzyskiwać stanu albo raportować sytuacji wyjątkowych.

Podatnością nie jest samo wystąpienie błędu. Pytanie bezpieczeństwa brzmi: jak aplikacja zachowuje się po błędzie albo nieprawidłowym stanie?

## Przykłady exceptional conditions

- missing, malformed albo unexpected input trafia do niebezpiecznej ścieżki,
- nieoczekiwane `null` albo `undefined`,
- awaria bazy, file storage albo external API,
- authorization service unavailable,
- session store failure,
- timeout albo client disconnect,
- failed cleanup,
- partial transaction,
- unsafe retry,
- duplicate request,
- race condition,
- resource exhaustion,
- sensitive error details ujawnione użytkownikowi.

## Expected validation failure kontra exceptional condition

Expected validation failure to kontrolowane odrzucenie błędnego inputu, np. brak pola, pusta wartość, `null`, zły typ, tablica zamiast stringa, unknown enum albo malformed ID. Zwykle powinno to zwrócić kontrolowane `4xx`, np. `400`.

Exceptional condition to nieprawidłowy stan, który przerywa processing albo sprawia, że aplikacja nie wie, co wydarzyło się dalej, np. database timeout, file-storage failure, authorization dependency unavailable, unexpected exception, client timeout po tym, jak serwer mógł już przetworzyć request, albo dwa requesty zmieniające ten sam state.

## A10 to nie każdy exception

`500` może być evidence of unhandled condition, ale nie jest automatycznie podatnością. Sprawdzam, czy odpowiedź ujawniła stack trace, SQL query, path, hostname, secret albo config; czy system kontynuował po failed security check; czy przyznał access bez potwierdzonej authorization; czy zostawił partial state; czy retry może zdublować side effect; czy resources zostały zwolnione.

## A09 kontra A10

A09 dotyczy visibility, detection, alertingu i response. A10 dotyczy unsafe behaviour during abnormal conditions.

```text
A10: authorization service fails -> sensitive action is denied -> state remains safe
A09: the failure is logged and alerted when repeated or suspicious
```

Logging unsafe failure nie czyni go bezpiecznym.

## A01 / A06 kontra A10

```text
Missing authorization check -> A01 / A06
Authorization check exists but dependency timeout causes allow -> A10 fail-open
```
