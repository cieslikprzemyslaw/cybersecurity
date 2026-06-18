# Ćwiczenie 05: Duplicate Submit and Race-Condition Basics

## Scenariusz

Dwa requesty są wysłane prawie w tym samym czasie:

```http
PATCH /api/assessments/asm_123/status

{
  "status": "completed"
}
```

Business invariant:

```text
Assessment może zostać completed tylko raz.
Jedno completion powinno utworzyć jeden completion event.
```

Problem flow:

```text
Request A czyta status = in-progress.
Request B czyta status = in-progress.
Request A aktualizuje status na completed i tworzy activity event.
Request B też aktualizuje status na completed i tworzy activity event.
```

Final state:

```text
assessment.status = completed
activity log zawiera dwa wpisy "Assessment completed"
```

## Shared state

Shared state to nie tylko ID. ID wskazuje wspólny zasób.

Shared state obejmuje:

- `assessment.status`,
- completion timestamp,
- activity log entries dla tego assessmentu,
- counters albo report state zależne od completion.

## Co zrozumiałem

Frontend disabled buttons zmniejszają przypadkowe double-clicki, ale nie egzekwują bezpieczeństwa.

Requesty nadal można wysłać przez:

- Burp Suite,
- OWASP ZAP,
- Postman,
- DevTools,
- scripts,
- drugą kartę przeglądarki,
- retry przez wolną sieć,
- albo mobile/browser retry behaviour.

## Lekcja A10

To można traktować jako A10, gdy aplikacja źle obsługuje abnormal timing/state condition:

```text
two concurrent requests -> same mutable state -> check-before-update race -> duplicate side effect
```

Może to być też business logic bug. Staje się security-relevant, gdy łamie integrity, access control, auditability, billing, workflow rules albo inne security invariants.

## Bezpieczne oczekiwane zachowanie

Backend powinien zachować invariant nawet wtedy, gdy concurrent requests przyjdą równocześnie.

Na moim obecnym poziomie nie muszę projektować pełnej strategii lockingu. Muszę rozpoznać wymaganie:

```text
Dwa concurrent completion requests nie mogą utworzyć dwóch completion side effects.
```

Bezpieczne wyniki mogą obejmować:

- jeden request kończy operację, a drugi zwraca aktualny completed state,
- jeden request kończy operację, a drugi jest bezpiecznie odrzucony,
- oba requesty zwracają safe result, ale istnieje tylko jeden completion event.

## Testy regresji

Przydatne testy:

- wyślij dwa concurrent `PATCH completed` requests dla tego samego assessmentu,
- sprawdź, że assessment kończy jako `completed`,
- sprawdź, że istnieje tylko jeden completion activity event,
- sprawdź, że nie ma duplicate notification/email/report-finalisation side effect,
- sprawdź, że UI pozostaje spójne po refresh,
- sprawdź, że frontend disabled button jest traktowany jako UX only, a nie jedyna kontrola.

## Frontend takeaway

Frontend powinien zapobiegać przypadkowym duplicate clicks dla UX, ale backend musi zachować workflow invariant. Jeżeli backend polega tylko na disabled button, kontrolę można obejść.
