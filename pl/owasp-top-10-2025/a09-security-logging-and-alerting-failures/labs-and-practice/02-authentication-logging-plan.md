# Authentication Logging and Alerting Plan

## Scenariusz

```text
3 failed logins
    -> 1 successful login
    -> 2 failed MFA attempts
    -> successful MFA
    -> authenticated session created
```

## Autorytatywne źródło

Auth Service powinien emitować autorytatywne eventy dotyczące hasła, MFA i utworzenia sesji. Frontend może dostarczać supporting telemetry, ale nie zna wiarygodnego rezultatu kontroli backendowej.

## Eventy

```text
authn_login_fail
authn_login_success
authn_mfa_fail
authn_mfa_success
session_created
```

Recovery i reset powinny mieć osobne eventy tylko wtedy, gdy konkretny workflow rzeczywiście wystąpił.

## Przykładowy `authn_login_fail`

```json
{
  "timestamp": "2026-06-16T12:00:00Z",
  "eventName": "authn_login_fail",
  "accountId": "usr_123",
  "sourceIp": "203.0.113.24",
  "userAgent": "Mozilla/5.0",
  "requestId": "req_456",
  "result": "failure",
  "reasonCode": "INVALID_CREDENTIALS"
}
```

Nie logujemy hasła, JWT, access/refresh tokenu, session cookie, MFA code, reset tokenu ani pełnego nagłówka `Authorization`.

## Reguła 1 — powtarzające się błędy

```text
Pattern: authn_login_fail dla jednego account ID
Threshold: 5
Window: 5 minut
Severity: WARN
Owner: security team albo on-call
Response: sprawdzenie źródeł prób, rate limiting, ryzyka konta i powiązanych sesji
```

## Reguła 2 — sukces po wielu błędach

```text
Pattern: authn_login_success po co najmniej 5 authn_login_fail
Correlation: to samo account ID
Window: 10 minut
Severity: HIGH; wyższe dla privileged account
Owner: security team albo on-call
Response: review MFA, sesji i aktywności po logowaniu; containment przy wysokiej pewności
```

## Wniosek

Pojedynczy event jest dowodem zdarzenia. Detekcja wymaga korelacji, progu i okna czasowego, a alert wymaga ownera i oczekiwanej reakcji.
