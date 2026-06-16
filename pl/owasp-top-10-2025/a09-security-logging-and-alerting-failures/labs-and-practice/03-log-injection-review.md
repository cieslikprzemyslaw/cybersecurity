# Log Injection Review

## Analizowany kod

```ts
logger.warn(
  `${new Date().toISOString()} LOGIN_FAILED username=${username} ip=${request.ip}`
);
```

## User-controlled values

- `username` pochodzi bezpośrednio z request body,
- `request.ip` może zależeć od konfiguracji proxy i nagłówków takich jak `X-Forwarded-For`,
- `password` również pochodzi z requestu, ale nie był logowany i nie powinien być.

## Problem

Log jest budowany jako zwykły string. `username` może wpływać na strukturę line-oriented logu.

```text
untrusted input
    -> interpolated text
    -> parser / viewer / SIEM
    -> misleading interpretation
```

## Możliwe skutki

- forged-looking second event,
- uszkodzona struktura,
- wrong actor albo severity,
- false positive,
- false negative,
- utrata integralności i accountability,
- utrudnione dochodzenie.

Atakujący w tym przykładzie nie musi nadpisywać wcześniejszego logu. Może sprawić, że nowy wpis będzie wyglądał jak kilka wpisów albo zostanie błędnie zinterpretowany.

## Bezpieczniejszy projekt

```ts
logger.warn("Authentication failed", {
  timestamp: new Date().toISOString(),
  eventName: "authn_login_fail",
  accountId,
  sourceIp: request.ip,
  requestId: request.id,
  result: "failure",
  reasonCode: "INVALID_CREDENTIALS",
});
```

Dodatkowo:

- normalizacja lub encoding control characters,
- limity długości pól,
- application-controlled metadata,
- brak password i innych sekretów,
- test parsera i viewerów downstream.

## Regression test

Test wysyła logowalną wartość ze znakami newline lub control characters i sprawdza:

- dokładnie jeden structured event,
- niezmieniony `eventName`, severity, result i reason code,
- wartość inputu pozostaje wewnątrz jednego pola,
- brak fałszywego drugiego eventu,
- brak hasła w zapisanych danych.

## Wniosek

Dane użytkownika mogą być wartością pola logu, ale nie powinny kontrolować struktury, typu, severity ani znaczenia eventu.
