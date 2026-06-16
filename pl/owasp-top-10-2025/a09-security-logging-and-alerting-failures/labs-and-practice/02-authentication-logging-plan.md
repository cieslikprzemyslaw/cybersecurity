# Plan Logowania i Alertowania dla Authentication

## Scenariusz

```text
trzy nieudane logowania
    -> udana weryfikacja hasła
    -> dwie nieudane próby MFA
    -> udane MFA
    -> utworzona uwierzytelniona sesja
```

## Źródło autorytatywne

Identity lub Auth Service powinien emitować autorytatywne eventy authentication, ponieważ weryfikuje credentiale, ukończenie MFA oraz stan sesji.

Frontend telemetry może rejestrować błędy UX albo request ID, ale nie może dowieść, że authentication się powiodło.

## Oczekiwane eventy

```text
authn_login_fail
authn_login_fail
authn_login_fail
authn_login_success
authn_mfa_fail
authn_mfa_fail
authn_mfa_success
session_created
```

## Przykładowy event failed-login

```json
{
  "timestamp": "2026-06-16T12:00:00Z",
  "eventName": "authn_login_fail",
  "accountId": "usr_123",
  "sourceIp": "203.0.113.24",
  "userAgent": "Mozilla/5.0",
  "requestId": "req_a87f",
  "result": "failure",
  "reasonCode": "INVALID_CREDENTIALS"
}
```

Przykład ma osiem pól. HTTP status albo route mogą być dodane, jeśli odpowiadają na konkretną potrzebę dochodzeniową, ale query string nie może zawierać credentiali.

## Wykluczane dane

Nie loguj:

- hasła,
- kodu MFA,
- session cookie,
- access lub refresh tokenu,
- reset tokenu,
- pełnego nagłówka `Authorization`,
- pełnego body requestu logowania.

## Reguła detekcji 1: powtarzające się failures

```text
Pattern: authn_login_fail
Correlation: to samo konto
Threshold: 5 failures
Window: 5 minutes
Severity: WARN
Owner: security team lub on-call owner
Response: review źródeł, rate limits, account risk i powiązanej aktywności
```

## Reguła detekcji 2: success po failures

```text
Pattern: authn_login_success po co najmniej 5 eventach authn_login_fail
Correlation: to samo konto
Window: 10 minutes
Severity: HIGH; wyższe dla kont uprzywilejowanych, jeśli polityka tego wymaga
Owner: security team lub on-call owner
Response: review MFA, session, source context i działań po logowaniu; zawrzyj, gdy confidence jest wysoka
```

## Korekta nauki

Moja pierwsza odpowiedź wymieniała login i MFA eventy, ale nie obejmowała lifecycle sesji ani nie definiowała pełnej reguły detekcji. Musiałem też rozdzielić pojedynczy event od skorelowanego wzorca.

## Główna myśl

> Auth Service emituje evidence. Reguła detekcji zamienia podejrzaną sekwencję tego evidence w actionable alert.
