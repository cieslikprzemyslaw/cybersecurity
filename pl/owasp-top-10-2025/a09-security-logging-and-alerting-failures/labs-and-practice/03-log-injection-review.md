# Review Log Injection

## Zbadany kod

```ts
app.post("/login", async (request, response) => {
  const { username, password } = request.body;
  const user = await authenticate(username, password);

  if (!user) {
    logger.warn(
      `${new Date().toISOString()} LOGIN_FAILED username=${username} ip=${request.ip}`
    );

    return response.status(401).json({ message: "Invalid credentials" });
  }

  logger.info(
    `${new Date().toISOString()} LOGIN_SUCCESS username=${username} ip=${request.ip}`
  );

  return response.status(200).json({ message: "Logged in" });
});
```

## Dane kontrolowane przez użytkownika

- `username` pochodzi bezpośrednio z body requestu.
- `password` też pochodzi z body requestu, ale nie jest wstawiane do pokazanego log line.
- `request.ip` zależy od konfiguracji Express i proxy. Przekazane nagłówki mogą być mylące, jeśli `trust proxy` jest ustawione nieprawidłowo.

## Problem z konstrukcją

Event jest składany jako plain string. Kontrolowana wartość jest wstawiana do tego samego tekstu, który definiuje granice eventu i nazwy pól.

```text
niezaufany username
    -> interpolowany line
    -> line parser / viewer / SIEM
```

## Potencjalny wpływ

Atakujący może być w stanie:

- sfałszować wpis wyglądający jak drugi event,
- uszkodzić strukturę rekordu,
- ukryć złośliwą aktywność,
- wywołać false positive albo false negative,
- przypisać aktywność niewłaściwemu aktorowi,
- wprowadzić w błąd analityka.

Ćwiczenie nie dowiodło, że istniejące zapisane rekordy mogły zostać nadpisane. Zwykle wymagałoby to dodatkowego dostępu albo innej podatności.

## Bezpieczniejszy design

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

Dodatkowe kontrole:

- strukturalna serializacja,
- limity długości pól,
- obsługa znaków kontrolnych,
- event name, severity, result i reason code kontrolowane przez aplikację,
- brak logowania hasła,
- bezpieczny downstream parsing i display.

## Test regresji

1. Wyślij username zawierający newline i znaki kontrolne.
2. Doprowadź do nieudanego uwierzytelnienia.
3. Zbierz wysłany event.
4. Potwierdź, że istnieje dokładnie jeden event.
5. Potwierdź, że event pozostaje `authn_login_fail` z oczekiwaną severity i result.
6. Potwierdź, że username jest bezpiecznie zawarty w jednym polu.
7. Potwierdź, że nie ma hasła ani dodatkowego sfałszowanego rekordu.

## Główna myśl

> Niezaufane dane mogą być wartością pola, ale nie mogą kontrolować struktury ani znaczenia security eventu.
