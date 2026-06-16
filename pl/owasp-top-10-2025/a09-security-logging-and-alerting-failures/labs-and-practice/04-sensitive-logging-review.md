# Review Sensitive Logging

## Scenariusz

Backend loguje udaną zmianę roli administratora z polami obejmującymi dane aktora, dane celu, source context, tokeny, body requestu, stare i nowe role oraz wynik.

## Ostateczna klasyfikacja

| Pole | Klasyfikacja | Uzasadnienie |
|---|---|---|---|
| `timestamp` | Log directly | Wymagany do porządkowania eventów i dochodzenia. |
| `eventName` | Log directly | Definiuje security event. |
| `actorUsername` | Mask, transform, or controlled use | Może być PII i może się zmieniać; preferowany jest internal ID. |
| `actorUserId` | Log directly | Niezbędny dowód tego, kto wykonał akcję. |
| `targetUserId` | Log directly | Niezbędny dowód tego, czyja rola została zmieniona. |
| `sourceIp` | Direct or transformed by policy | Przydatny kontekst, ale niepewna tożsamość. |
| `userAgent` | Controlled use | Przydatny w niektórych dochodzeniach, ale nie zawsze konieczny. |
| `sessionCookie` | Do not log | Sekret uwierzytelniający. |
| `accessToken` | Do not log | Sekret uwierzytelniający; hashowanie nie jest potrzebne dla tego eventu. |
| `requestId` | Log directly | Bezpieczna korelacja między usługami. |
| `errorCode` | Log directly when safe | Użyj application-controlled reason code, a nie stack trace. |
| `requestBody` | Do not log | Może zawierać hasła, tokeny, tekst uzasadnienia i zbędne dane. |
| `previousRole` | Log directly | Wymagane do odtworzenia zmiany. |
| `newRole` | Log directly | Wymagane do odtworzenia zmiany. |
| `result` | Log directly | Pokazuje, czy akcja się powiodła. |

## Minimalny audit evidence

```text
actorUserId
targetUserId
previousRole
newRole
timestamp
result
```

`eventName` i `requestId` czynią rekord bardziej czytelnym i łatwiejszym do korelacji.

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

CSRF token nie powinien być logowany, aby dowieść, czy walidacja zadziałała. Zamiast tego loguj wynik kontroli:

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

## Korekta nauki

Początkowo usunąłem actor i target IDs, jednocześnie uznając hash access tokenu za akceptowalny. Korekta była taka, że internal IDs aktora i celu są niezbędnym dowodem auditowym, a sam access token nie jest potrzebny i nie powinien być zapisywany.
