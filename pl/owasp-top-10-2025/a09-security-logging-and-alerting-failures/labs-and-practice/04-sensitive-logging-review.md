# Sensitive Logging Review

## Scenariusz

Backend zapisuje audit event zmiany roli użytkownika. Pierwotny obiekt zawierał zarówno potrzebny kontekst, jak i sekrety.

## Klasyfikacja pól

| Pole | Klasyfikacja | Uzasadnienie |
|---|---|---|
| `timestamp` | Logować bezpośrednio | Potrzebny do osi czasu. |
| `eventName` | Logować bezpośrednio | Określa typ eventu. |
| `actorUsername` | Maskować lub logować wyjątkowo | Może być PII; internal ID jest stabilniejszy. |
| `actorUserId` | Logować bezpośrednio | Kluczowy dowód, kto wykonał akcję. |
| `targetUserId` | Logować bezpośrednio | Kluczowy dowód, czyje uprawnienia zmieniono. |
| `sourceIp` | Logować bezpośrednio lub z kontrolą | Przydatny kontekst, ale nie dowód tożsamości. |
| `userAgent` | Logować zależnie od potrzeby | Wspiera dochodzenie, ale nie zawsze jest konieczny. |
| `sessionCookie` | Nie logować | Sekret uwierzytelniający. |
| `accessToken` | Nie logować | Sekret uwierzytelniający; hash nie jest potrzebny. |
| `requestId` | Logować bezpośrednio | Bezpieczna korelacja. |
| `errorCode` | Logować, jeśli kontrolowany | Używać safe reason code zamiast stack trace. |
| `requestBody` | Nie logować | Może zawierać hasło, tokeny i zbędne dane. |
| `previousRole` | Logować bezpośrednio | Potrzebne do odtworzenia zmiany. |
| `newRole` | Logować bezpośrednio | Potrzebne do odtworzenia zmiany. |
| `result` | Logować bezpośrednio | Pokazuje, czy akcja się powiodła. |

## Minimalny dowód auditowy

```text
actorUserId
targetUserId
previousRole
newRole
timestamp
result
```

`eventName` i `requestId` poprawiają czytelność i korelację.

## Bezpieczny event

```ts
logger.warn("User role changed", {
  timestamp: new Date().toISOString(),
  eventName: "privilege_permissions_changed",
  actorUserId: authenticatedUser.id,
  targetUserId: request.body.userId,
  previousRole,
  newRole: request.body.role,
  result: "success",
  requestId: request.id,
});
```

## Wyjaśnienie CSRF

Nie trzeba logować CSRF tokenu, aby udowodnić wynik walidacji. Logujemy rezultat kontroli:

```json
{
  "eventName": "csrf_validation_fail",
  "actorUserId": "usr_123",
  "requestId": "req_456",
  "route": "/api/users/usr_987/role",
  "method": "PATCH",
  "result": "failure",
  "reasonCode": "TOKEN_MISSING_OR_INVALID"
}
```

## Korekta mojego rozumienia

Początkowo usunąłem actor i target IDs, a hash access tokenu uznałem za dopuszczalny. Korekta: internal IDs są potrzebnym dowodem audytowym, natomiast access token nie jest potrzebny i nie powinien być zapisywany.
