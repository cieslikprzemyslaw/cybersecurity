# Frontend i API Logging

## Właściwa rola frontend telemetry

Frontend logging nie jest bezużyteczny. Może wspierać:

- JavaScript error reporting,
- diagnostykę failed resources i API calls,
- performance monitoring,
- analizę UX i kompatybilności przeglądarek,
- korelację z backend requests,
- Content Security Policy reports,
- wykrywanie nietypowego client-side behaviour.

Jednocześnie przeglądarka jest kontrolowana przez użytkownika. Client telemetry można zmienić, zablokować, odtworzyć albo sfabrykować.

## Właściciel autorytatywnego eventu

Komponent egzekwujący kontrolę powinien zwykle emitować autorytatywny security event.

| Decyzja bezpieczeństwa | Główne autorytatywne źródło |
|---|---|
| Hasło zaakceptowane albo odrzucone | Identity/Auth Service |
| MFA ukończone albo odrzucone | Identity/Auth Service |
| Sesja utworzona albo unieważniona | Session/Auth Service |
| Authorization allowed albo denied | API lub usługa egzekwująca dostęp |
| Zmiana roli | Administrative backend service |
| Plik accepted, rejected albo quarantined | Upload validation/malware-scanning service |
| Eksport wrażliwych danych | Business service wykonujący eksport |
| Rate limit applied | Gateway, middleware albo usługa egzekwująca limit |

Pomocnicze źródła:

- WAF,
- API gateway,
- web server,
- database audit logs,
- platform logs,
- frontend error telemetry.

Pomocniczy dowód nie powinien zastępować eventu aplikacji, gdy to aplikacja najlepiej zna aktora, cel, akcję i rezultat.

## Nie ujawniaj sekretów w browser logs

Unikaj:

```ts
console.log("token", accessToken);
console.log("login payload", formValues);
console.log("API response", responseBody);
```

Ryzyka:

- lokalny dostęp przez DevTools,
- browser extensions,
- screen sharing i support captures,
- third-party telemetry ingestion,
- nadmierny retention,
- dostęp zbyt wielu pracowników.

## Error tracking i data minimisation

Przed wysłaniem eventu do frontend error tracker sprawdź:

- URL i query string,
- request headers,
- cookies,
- form fields,
- DOM snapshots,
- API response data,
- user identifiers,
- CMS content,
- source maps i stack traces.

Stosuj allowlists, redaction hooks, sampling, retention limits i access controls. Nie polegaj wyłącznie na domyślnym scrubbingu dostawcy.

## Wzorzec korelacji

```text
browser request with request ID
    -> API receives request ID or creates a trusted one
    -> backend emits authoritative security event
    -> frontend error telemetry records the same non-secret ID
    -> monitoring correlates both sources
```

Request ID wspiera investigation, ale nie przyznaje authentication ani authorization.

## Przykład eventu API

```ts
logger.warn("Authorization denied", {
  timestamp: new Date().toISOString(),
  eventName: "authz_fail",
  actorUserId: request.auth.userId,
  affectedResource: `company:${request.params.companyId}`,
  action: "company.update",
  result: "failure",
  reasonCode: "INSUFFICIENT_PERMISSION",
  requestId: request.id,
  sourceIp: request.ip,
});
```

Nie dodawaj:

- access tokenu,
- pełnych cookies,
- całego request body,
- zawartości wrażliwego zasobu,
- stack trace w standardowym security evencie.

## Obsługa błędów na produkcji

User-facing response i internal event mają inne cele.

Odpowiedź dla użytkownika:

```json
{
  "message": "The request could not be completed."
}
```

Wewnętrzny event:

```json
{
  "eventName": "security_control_error",
  "requestId": "req_123",
  "service": "permissions-api",
  "result": "failure",
  "reasonCode": "POLICY_ENGINE_UNAVAILABLE"
}
```

Ogólny komunikat dla użytkownika nie usprawiedliwia pustego logu wewnętrznego. Szczegółowy event wewnętrzny nie usprawiedliwia ujawniania implementacji użytkownikowi.
