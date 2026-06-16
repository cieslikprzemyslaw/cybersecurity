# A09 — Podsumowanie praktyki

## Co przejrzałem

- application-level security events,
- event naming dla authentication i MFA,
- korelację i alert thresholds,
- log injection w ręcznie składanym Node/Express log line,
- bezpieczne pola dla audit eventu zmiany roli,
- frontend telemetry kontra autorytatywne evidence backendu.

## Główny detection gap

Najważniejszą luką nie był wyłącznie brak logów. Była nią sytuacja, w której eventy istnieją, ale żadna reguła ich nie koreluje, alert nie dociera do ownera albo nie istnieje proces response.

## Główna lekcja o sensitive data

Dochodzenie potrzebuje dowodu aktora, celu, akcji, czasu i rezultatu. Nie potrzebuje haseł, access tokens, session cookies, MFA codes ani pełnych request bodies.

## Główna lekcja log injection

String interpolation pozwala niezaufanej wartości wpływać na strukturę wpisu. Bezpieczniejszy design korzysta ze structured events, application-controlled metadata, limitów pól i testów newline/control characters.

## Wymagania bezpieczeństwa

- Authentication successes i failures muszą tworzyć structured security events.
- Zmiana roli musi tworzyć audit event z aktorem, celem, poprzednią rolą, nową rolą, timestampem i wynikiem.
- Logi nie mogą zawierać authentication secrets.
- User-controlled values nie mogą kontrolować struktury, nazwy, severity, result ani reason code eventu.
- Powtarzające się failures i success-after-failure muszą mieć zdefiniowane reguły detekcji.
- High-severity alerts muszą mieć ownera i playbook.
- Security logs muszą być chronione przed nieautoryzowanym odczytem, zmianą i usunięciem.

## Końcowa ocena

**PASS — Frontend Developer przechodzący do AppSec**

Potrafię podczas review zapytać:

```text
what happened?
what was logged?
what was excluded?
what pattern should alert?
who owns it?
what evidence proves the flow works?
```
