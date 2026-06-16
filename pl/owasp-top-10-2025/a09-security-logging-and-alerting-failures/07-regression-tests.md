# A09 — Testy Regresji

Wybierz testy istotne dla analizowanej funkcji.

## Authentication i session events

### Udane uwierzytelnienie

Wykonaj prawidłowe logowanie.

Oczekiwane:

- jeden event `authn_login_success`,
- prawidłowe account ID, timestamp, result, service i request ID,
- brak hasła, tokenu, cookie i pełnego request body.

### Nieudane uwierzytelnienie

Wyślij nieprawidłowe credentials.

Oczekiwane:

- jeden event `authn_login_fail`,
- bezpieczny reason code,
- ogólny komunikat dla użytkownika,
- brak account enumeration przez response lub ujawniony log.

### Detekcja wielu niepowodzeń

Wygeneruj zdefiniowaną liczbę błędów w time window reguły.

Oczekiwane:

- pojedyncze eventy pozostają dostępne,
- correlation rule dopasowuje się przy ustalonym progu,
- powstaje jeden użyteczny alert zamiast alertu dla każdego eventu,
- alert zawiera konto, source context, zakres czasu i nazwę reguły.

### Success after failures

Wygeneruj serię błędów, a następnie udane logowanie.

Oczekiwane:

- success event zostaje skorelowany z wcześniejszymi failures,
- severity rośnie zgodnie z polityką,
- alert trafia do wskazanego ownera,
- playbook wskazuje sesję i post-login activity do review.

### MFA i recovery

Przetestuj:

- MFA failure,
- wielokrotne MFA failure,
- MFA recovery,
- password-reset request i completion,
- invalid albo reused reset token.

Oczekiwane:

- osobne event types i results,
- brak MFA code, recovery code i reset tokenu w logach,
- high-risk sequences generują oczekiwaną detekcję.

### Unieważniona sesja

Użyj expired albo revoked session.

Oczekiwane:

- request odrzucony,
- `session_use_after_expire` albo równoważny event,
- wielokrotne użycie można korelować i alertować.

## Authorization i audit events

### Audit zmiany roli

Zmień rolę użytkownika.

Oczekiwany event zawiera:

```text
timestamp
eventName
actorUserId
targetUserId
previousRole
newRole
result
requestId
```

Oczekiwane wykluczenia:

```text
password
sessionCookie
accessToken
complete requestBody
```

### Nieudana zmiana roli

Spróbuj wykonać zmianę bez uprawnienia.

Oczekiwane:

- authorization denial event,
- brak zmiany roli,
- bezpieczne wskazanie targetu i attempted action,
- brak wrażliwej zawartości rekordu.

### High-risk action bez audit eventu

W środowisku testowym tymczasowo usuń albo uszkodź emission eventu.

Oczekiwane:

- automated test nie przechodzi,
- deployment lub quality gate wykrywa brak wymaganego audytu.

## Log injection

### Newline i control characters

Wyślij username, filename lub header zawierający newline i control characters.

Oczekiwane:

- dokładnie jeden poprawny structured event,
- input pozostaje w jednym polu,
- event name, severity, result i reason code pozostają application-controlled,
- brak fałszywego drugiego wpisu,
- downstream parser i viewer pozostają bezpieczne.

### Nadmiernie długie pole

Wyślij bardzo długą wartość przeznaczoną do logowania.

Oczekiwane:

- udokumentowane truncation albo rejection,
- brak awarii storage lub parsera,
- zachowanie oryginalnego event type i metadata.

## Sensitive-data leakage

### Secret scanning przechwyconych logów

Wykonaj login, password reset, MFA, upload i role change, a następnie przejrzyj eventy.

Oczekiwany brak:

- passwords,
- session IDs i cookies,
- access i refresh tokens,
- reset tokens,
- MFA secrets i codes,
- API keys,
- private keys,
- pełnych `Authorization` headers.

### Redaction w frontend telemetry

Wywołaj client error, gdy strona zawiera wrażliwe form data albo API response.

Oczekiwane:

- wyłącznie zatwierdzone metadata i correlation ID,
- brak sensitive DOM snapshot, form value, tokenu, cookie i response body.

## Collection, availability i integrity

### Awaria collectora albo sieci

Zasymuluj utratę remote collectora lub network connectivity.

Oczekiwane:

- security control nie działa fail open,
- event loss albo buffering zachowuje się zgodnie z polityką,
- powstaje operational signal o degraded logging,
- aplikacja nie ujawnia użytkownikowi internal errors.

### Wyczerpanie storage

Zasymuluj wyczerpanie dysku albo ingestion capacity.

Oczekiwane:

- aplikacja zachowuje zdefiniowane availability behaviour,
- awaria loggingu jest wykryta,
- eventy nie znikają bez operational signal.

### Nieautoryzowany odczyt logów

Spróbuj odczytać logi jako nieuprawniony user albo service.

Oczekiwane:

- access denied,
- próba jest logowana tam, gdzie to właściwe,
- eventy nie są dostępne przez publiczną ścieżkę webową.

### Próba modyfikacji albo usunięcia

Spróbuj zmienić lub usunąć chroniony audit record.

Oczekiwane:

- operacja odrzucona albo wykryta,
- dowód próby zachowany oddzielnie,
- alert zgodny z polityką.

### Spójność timestampów

Wygeneruj eventy w istotnych usługach.

Oczekiwane:

- ISO 8601 i UTC offsets,
- zrozumiała kolejność i clock differences,
- rozdzielenie event time i ingestion time tam, gdzie potrzebne.

## Alert delivery i jakość

### Awaria kanału alertów

Zasymuluj failed notification channel.

Oczekiwane:

- delivery failure zostaje wykryty,
- fallback albo escalation path działa zgodnie z polityką,
- detection event pozostaje zapisany.

### Duplicate alerts

Wygeneruj wiele eventów dotyczących tego samego incydentu.

Oczekiwane:

- deduplication lub grouping działa,
- dowody zostają zachowane bez przeciążenia ownera.

### DAST i security testing

Uruchom autoryzowany scan albo controlled test, który powinien spełnić regułę.

Oczekiwane:

- eventy są klasyfikowane jako authorised testing, jeśli to potrzebne,
- nie są po cichu pomijane,
- reguła i alert nadal działają,
- evidence dokumentuje pełną ścieżkę event-to-response.
