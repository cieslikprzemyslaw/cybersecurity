# Podsumowanie Praktyki A09

## Co przejrzałem

- application-level security events,
- nazewnictwo eventów authentication i MFA,
- correlation i progi alertów,
- log injection w log line zbudowanym ze stringów w Node/Express,
- bezpieczne pola dla audit eventu zmiany roli uprzywilejowanej,
- frontend telemetry versus autorytatywny backendowy dowód.

## Główna luka detekcji

Najbardziej widoczna luka nie dotyczyła wyłącznie brakujących logów. Chodziło też o sytuację, w której eventy istniały, ale żadna reguła ich nie korelowała, alert nie trafiał do właściciela albo nie istniał proces reakcji.

## Najważniejsza lekcja o danych wrażliwych

Do dochodzenia potrzebny jest dowód aktora, celu, akcji, czasu i wyniku. Nie są potrzebne hasła, access tokeny, session cookies, kody MFA ani pełne body requestów.

## Najważniejsza lekcja o log injection

Interpolacja stringów pozwalała kontrolowanej wartości wpłynąć na strukturę log line. Bezpieczniejszy design używa structured events, application-controlled metadata, limitów pól i testów regresji przeciwko newline oraz znakom kontrolnym.

## Wymagania bezpieczeństwa, które wyniknęły

- Authentication successes i failures muszą tworzyć structured security events.
- Privileged role changes muszą tworzyć audit event zawierający actor, target, previous role, new role, timestamp i result.
- Logi nie mogą zawierać authentication secrets.
- Wartości kontrolowane przez użytkownika nie mogą sterować strukturą eventu, nazwą, severity, wynikiem ani reason code.
- Powtarzające się failures authentication i wzorzec success-after-failure muszą mieć zdefiniowane reguły detekcji.
- High-severity alerts muszą mieć właściciela i playbook reakcji.
- Security logs muszą być chronione przed nieautoryzowanym dostępem, modyfikacją i usunięciem.

## Ocena końcowa

**PASS — Frontend Developer przechodzący do AppSec**

Potrafię przejrzeć prostą funkcję i zadać pytania:

```text
co się wydarzyło?
co zostało zalogowane?
co zostało wykluczone?
jaki wzorzec powinien uruchomić alert?
kto jest właścicielem?
jaki dowód potwierdza, że przepływ działa?
```
