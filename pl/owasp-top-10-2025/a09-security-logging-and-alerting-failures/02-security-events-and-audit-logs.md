# Security Events i Audit Logs

## Wybór eventów według pytania bezpieczeństwa

Więcej logowania nie oznacza automatycznie lepszej ochrony. Każdy event powinien pomagać odpowiedzieć na konkretne pytanie:

- Czy ktoś atakuje konto?
- Czy dostęp do chronionego zasobu został prawidłowo odrzucony?
- Kto zmienił uprawnienia użytkownika?
- Czy ponownie użyto nieważnego lub unieważnionego tokenu?
- Czy plik nie przeszedł walidacji lub malware scan?
- Czy utworzono wrażliwy eksport?
- Czy użytkownik ominął oczekiwaną kolejność procesu biznesowego?

## Eventy, które często warto logować

### Authentication i session

- udane i nieudane logowanie,
- udane i nieudane MFA,
- recovery lub reset MFA,
- request password reset, walidacja tokenu, ukończenie i failure,
- utworzenie, odnowienie, wygaśnięcie, logout i invalidation sesji,
- ponowne użycie expired, revoked albo invalid tokenu,
- udane logowanie po serii niepowodzeń.

### Authorization i privileges

- authorization denial,
- zmiana roli lub uprawnienia,
- akcje administracyjne,
- utworzenie albo usunięcie privileged usera,
- użycie konta wspólnego, domyślnego, awaryjnego lub break-glass,
- próby bezpośredniego dostępu do chronionych obiektów.

### Dane i logika biznesowa

- dostęp do bardzo wrażliwych rekordów,
- sensitive create/read/update/delete/import/export,
- high-value transaction,
- nieoczekiwana zmiana stanu,
- wykonanie kroku poza kolejnością,
- próba przekroczenia limitu biznesowego,
- akcja, która nie ma sensu w danym kontekście biznesowym.

### Walidacja i system

- server-side input-validation failure,
- output-validation failure,
- nieoczekiwana metoda HTTP,
- podejrzany błąd walidacji JWT,
- uruchomienie rate limit,
- wynik walidacji uploadu i malware scan,
- deserialization failure,
- błąd TLS backendu lub walidacji certyfikatu,
- błąd kontroli bezpieczeństwa,
- wyłączenie loggingu lub monitoringu,
- startup, shutdown, crash i inicjalizacja logowania.

## Loguj istotne sukcesy i porażki

Logowanie wyłącznie niepowodzeń tworzy niepełny obraz. Istotne sukcesy to między innymi:

- udane logowanie po wielu błędach,
- ukończony password reset,
- udana zmiana roli,
- udany eksport wrażliwych danych,
- udane użycie emergency access,
- udane usunięcie administracyjne.

Udana akcja może być najmocniejszym sygnałem, że wcześniejsza próba ataku ostatecznie się powiodła.

## Przydatna struktura eventu

Nie każdy event wymaga każdego pola. Należy wybrać minimum potrzebne do monitoringu i dochodzenia.

```text
timestamp
eventName
severity
application / service / version
environment
actorUserId
targetUserId lub affectedResource
action
result
safeReasonCode
requestId / correlationId / interactionId
sourceIp
userAgent
httpMethod / route / statusCode
securityControlResult
confidence
responseTaken
```

Event powinien zwykle odpowiadać na:

```text
when + where + who + what + result
```

## Event type, result i severity to osobne pola

```json
{
  "eventName": "authn_login_fail",
  "result": "failure",
  "severity": "WARN"
}
```

- `eventName` opisuje, co się wydarzyło.
- `result` opisuje rezultat.
- `severity` opisuje znaczenie eventu i priorytet reakcji w danym kontekście.

`FAILED` jest rezultatem, a nie uniwersalnym poziomem severity.

## Przykłady spójnego vocabulary

```text
authn_login_success
authn_login_successafterfail
authn_login_fail
authn_login_fail_max
authn_login_lock
authn_password_change
authn_password_change_fail
authn_token_created
authn_token_revoked
authn_token_reuse
authz_fail
authz_change
authz_admin
excess_rate_limit_exceeded
upload_validation
input_validation_fail
malicious_sqli
malicious_direct_reference
privilege_permissions_changed
sensitive_create
sensitive_read
sensitive_update
sensitive_delete
session_created
session_renewed
session_expired
session_use_after_expire
sys_startup
sys_shutdown
sys_crash
sys_monitor_disabled
user_created
user_updated
user_archived
user_deleted
```

Spójne nazwy ułatwiają detekcję pomiędzy usługami, ale nie zastępują pełnej polityki. Organizacja nadal potrzebuje:

- udokumentowanych nazw i typów pól,
- spójnych timestampów,
- reguł severity,
- progów i time windows,
- właścicieli alertów i playbooków.

## Przykładowe poziomy severity

OWASP pokazuje między innymi:

- `INFO` dla oczekiwanych, ale istotnych dla bezpieczeństwa zdarzeń,
- `WARN` dla podejrzanych, politycznie istotnych albo administracyjnie wrażliwych działań,
- `CRITICAL` dla high-confidence i high-risk indicators.

Nie są to uniwersalne mapowania. Severity powinno uwzględniać:

- wrażliwość zasobu,
- rezultat eventu,
- confidence,
- częstotliwość,
- poziom uprawnień konta,
- wpływ biznesowy,
- wymaganą szybkość reakcji.

## Przykład audit eventu — zmiana roli

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

Wewnętrzne identyfikatory aktora i celu są dowodem, a nie sekretem. Hasło, access token, session cookie i pełne request body nie są potrzebne, aby udowodnić zmianę roli.

## Integralność audit logów

Wartościowe audit trails powinny być chronione przed nieautoryzowanym:

- odczytem,
- modyfikacją,
- usunięciem,
- truncation,
- zmianą kolejności,
- manipulacją czasem.

Możliwe kontrole:

- append-oriented albo immutable storage,
- ścisłe uprawnienia zapisu i odczytu,
- central collection,
- chroniony transport,
- retention i backup policies,
- monitoring zatrzymania logowania lub prób usunięcia,
- rozdzielenie aplikacji zapisującej logi od osób lub usług, które je odczytują i administrują nimi.
