# Sensitive Data i Log Injection

## Logi są security assetem i attack surface

Logi wspierają obronę, ale mogą również być celem ataków.

### Poufność

Nieuprawniony czytelnik może uzyskać:

- dane osobowe lub zdrowotne,
- hasła lub tokeny,
- wewnętrzne ścieżki i hostnames,
- database connection details,
- informacje handlowo wrażliwe,
- zachowanie security controls.

### Integralność

Atakujący może:

- sfałszować wpis,
- zmienić pola,
- wstrzyknąć dodatkowe linie,
- usunąć dowody,
- zmienić zarejestrowanego aktora,
- uszkodzić parser albo downstream viewer.

### Dostępność

Atakujący może:

- zalać system logami,
- wyczerpać miejsce na dysku lub ingestion quota,
- uniemożliwić zapis kolejnych eventów,
- przeciążyć collectory albo analytics,
- wykorzystać kosztowne synchroniczne logowanie do pogorszenia wydajności aplikacji.

### Accountability

Atakujący może blokować zapis, uszkodzić dowody albo spowodować zapisanie niewłaściwej tożsamości, aby nie dało się wiarygodnie przypisać odpowiedzialności.

## Dane, których normalnie nie należy logować

- plaintext passwords,
- session cookies i session identifiers,
- access i refresh tokens,
- password-reset tokens,
- MFA secrets, codes i recovery codes,
- API keys,
- private cryptographic keys,
- pełne nagłówki `Authorization`,
- database connection strings,
- pełne dane kart płatniczych,
- zbędne wrażliwe dane osobowe i zdrowotne,
- pełne request/response bodies bez zatwierdzonego wymagania,
- dumpy environment variables,
- confidential CMS content,
- source code.

## Dane wymagające kontrolowanego traktowania

Zależnie od potrzeby dochodzeniowej i podstawy prawnej:

- username i e-mail,
- source IP,
- user agent,
- internal hostname,
- file path,
- ograniczone szczegóły błędu,
- masked account reference.

Możliwe kontrole:

- internal IDs zamiast display names,
- masking albo pseudonymisation,
- restricted exceptional logs,
- krótszy retention,
- oddzielne chronione storage,
- role-based access i audyt odczytu logów.

Hashowanie nie sprawia automatycznie, że sekret jest bezpieczny. Hash może pozostać stałym identyfikatorem, umożliwiać guessing dla przewidywalnych wartości i być niepotrzebny, gdy istnieje niezależny correlation ID.

## Korelacja bez sekretów uwierzytelniających

Preferuj:

```text
requestId
correlationId
interactionId
server-generated session reference
```

Nie używaj prawdziwego access tokenu albo session cookie jako wartości korelacyjnej.

## Model log injection

```text
untrusted input
    -> log message
    -> parser / viewer / SIEM
    -> misleading or dangerous interpretation
```

Potencjalnie kontrolowane pola:

- username,
- filename,
- query parameter,
- header,
- form value,
- user-agent string,
- error text z innego systemu.

Newline i control characters mogą:

- utworzyć wyglądający wiarygodnie drugi wpis,
- ukryć prawdziwy event,
- zepsuć JSON albo line-oriented parsing,
- przypisać złą severity lub aktora,
- powodować false positives lub false negatives,
- wprowadzić analityka w błąd.

## Niebezpieczny wzorzec

```ts
logger.warn(
  `${new Date().toISOString()} LOGIN_FAILED username=${username} ip=${request.ip}`
);
```

Wartość kontrolowana przez użytkownika jest wstawiana do struktury ręcznie składanego log line.

## Bezpieczniejszy wzorzec

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

Właściwości bezpieczeństwa:

- aplikacja kontroluje `eventName`, severity, result i reason code,
- user input pozostaje wartością pola,
- logger wykonuje structured serialisation,
- długość pól i control characters mogą być walidowane lub normalizowane,
- hasło nie jest logowane.

Structured logging zmniejsza ryzyko, ale nie jest magiczną ochroną. Biblioteka, transport, parser, viewer i integracje muszą bezpiecznie zachować strukturę.

## Główne kontrole

- structured logging,
- brak konkatenacji z niezaufanymi wartościami,
- encoding, escaping, odrzucenie lub normalizacja unsafe control characters zależnie od formatu,
- maksymalne długości pól,
- application-controlled event names i reason codes,
- test downstream parserów i viewerów,
- ograniczenie dostępu do logów,
- ochrona zapisanych rekordów przed zmianą i usunięciem.

## Oczekiwany test regresji

Wartość z newline lub control characters powinna utworzyć dokładnie jeden poprawny event. Nie może zmienić:

- event type,
- severity,
- result,
- reason code,
- granic pól,
- liczby zapisanych eventów.
