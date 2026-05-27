# A02 Testy regresji: Security Misconfiguration

## Cel

Testy regresji powinny udowodnić, że poprawka konfiguracji i obsługi błędów działa oraz że verbose debug information nie wróci później.

Dla Security Misconfiguration nie wystarczy sprawdzić, że invalid input zwraca błąd. Testy powinny też potwierdzać, że odpowiedź błędu nie ujawnia wewnętrznych szczegółów implementacji ani wrażliwych wartości.

## Scenariusz docelowy

Ukończone ćwiczenie pokazało User Management API, w którym:

- valid numeric user IDs zwracały normalną odpowiedź,
- nienumeryczny user ID wywoływał verbose debug response,
- odpowiedź ujawniała traceback, wewnętrzną ścieżkę pliku, nazwę funkcji, numer linii, typ wyjątku i wrażliwą flagę labową.

Poniższe testy regresji mają zapobiec temu patternowi.

Drugi ukończony lab pokazał publiczny endpoint:

```http
GET /cgi-bin/phpinfo.php
```

który zwracał stronę `phpinfo()` i ujawniał konfigurację oraz `SECRET_KEY`. Testy powinny więc obejmować także brak publicznych debug pages i brak sekretów w odpowiedziach.

## Positive tests

### 1. Valid numeric user ID nadal działa

```gherkin
Given the API is running with production-safe configuration
When a user sends GET /api/user/123
Then the response should be successful
And the response should return the expected user response structure
And the response should not include debug information
```

Oczekiwany bezpieczny kształt odpowiedzi:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

### 2. Frontend nadal obsługuje normalne odpowiedzi API

```gherkin
Given the frontend consumes the user API
When the API returns a valid user response
Then the UI should render the expected user state
And no debug-only fields should be required by the frontend
```

## Negative tests

### 3. Nienumeryczny user ID nie ujawnia debug information

```gherkin
Given the API is running with production-safe configuration
When a user sends GET /api/user/admin
Then the response status should be 400
And the response body should contain a generic error message
And the response body should not contain traceback information
And the response body should not contain internal file paths
And the response body should not contain exception types
And the response body should not contain secrets
```

Przykładowa bezpieczna odpowiedź:

```json
{
  "error": "Invalid user ID"
}
```

### 4. Inne invalid values są obsługiwane bezpiecznie

Wartości testowe:

```text
abc
admin
-1
1.5
999999999999999999999999
%27
null
undefined
```

Dla każdej wartości:

```gherkin
When the invalid value is used as the user ID
Then the API should return a controlled client-facing error
And the response should not expose internal implementation details
```

### 5. Nieoczekiwane metody nie ujawniają debug information

```gherkin
Given the user endpoint only supports expected methods
When a user sends an unsupported method to /api/user/123
Then the response should be controlled
And the response should not include raw framework or server error output
```

## Content checks

Automatyczne testy powinny sprawdzać, czy publiczne odpowiedzi nie zawierają niebezpiecznych stringów.

Odpowiedź nie powinna zawierać:

```text
Traceback
Exception
ValueError
TypeError
/app/
app.py
line 
debug_info
DATABASE_URL
API_KEY
JWT_SECRET
SECRET_KEY
TOKEN
flag
phpinfo()
PHP Version
```

Te checki nie są pełnym rozwiązaniem bezpieczeństwa, ale są użytecznymi regression guards dla tej konkretnej klasy podatności.

## Debug endpoint checks

### 6. `phpinfo()` nie jest publicznie dostępne

```gherkin
Given the application is running with production-safe configuration
When a user sends GET /cgi-bin/phpinfo.php
Then the response should be 404 or 403
And the response body should not contain phpinfo output
And the response body should not contain SECRET_KEY
And the response body should not contain environment variables
```

Preferowane zachowanie produkcyjne to `404 Not Found`, bo debug file nie powinien być wdrożony.

Jeśli diagnostyka jest celowo wspierana, endpoint musi być ograniczony do zaufanego/admin/internal access.

### 7. HTML source nie ujawnia debug links

Publiczny HTML nie powinien zawierać:

```text
/cgi-bin/phpinfo.php
/phpinfo.php
/debug
/config
/server-status
/actuator
```

Ten test ogranicza przypadkowe ujawnienie ścieżek, ale sam nie wystarcza. Backendowy endpoint musi być usunięty lub zablokowany nawet wtedy, gdy link zniknie z HTML.

### 8. Production artifact nie zawiera znanych debug files

Build/deployment check powinien failować, jeśli artifact produkcyjny zawiera:

```text
phpinfo.php
debug.php
test.php
info.php
.env
.env.production
backup.zip
database.sql
dump.sql
```

## Weryfikacja zachowania

Ten problem jest głównie information disclosure, więc może nie być zmiany stanu bazy danych do weryfikacji.

Testy powinny jednak potwierdzać:

- API pozostaje dostępne,
- valid input nadal działa,
- invalid input zwraca kontrolowany błąd,
- debug-only fields nie są zwracane,
- wrażliwe wartości nie pojawiają się w client-facing responses,
- frontend error rendering pozostaje przyjazny dla użytkownika.

## Weryfikacja server-side logging

Szczegółowe informacje błędów mogą być użyteczne dla developerów, ale muszą zostać server-side.

Pomysł na test:

```gherkin
Given a user submits invalid input
When the API handles the error
Then the client receives a generic error
And the server logs contain enough detail for debugging
And the logs are not exposed through the API response
```

Log powinien być chroniony i dostępny tylko dla uprawnionych osób operacyjnych.

## Testy konfiguracji środowiska

Dodaj checki dla konfiguracji specyficznej dla środowiska:

```gherkin
Given the application is deployed to production
Then debug mode should be disabled
And development error pages should not be exposed
And production error handlers should be active
```

Przykłady automatycznych checków:

- assert `DEBUG=false` lub odpowiednik,
- assert framework debug pages są wyłączone,
- assert raw exception handlers nie są aktywne,
- assert production configuration jest załadowana,
- assert test/development endpoints nie są publicznie dostępne.

## Manual verification

Kroki manualnej weryfikacji:

1. Wyślij poprawny request, na przykład `GET /api/user/123`.
2. Potwierdź, że normalna odpowiedź nadal działa.
3. Wyślij invalid requests, na przykład `GET /api/user/admin`, `GET /api/user/abc` i `GET /api/user/%27`.
4. Potwierdź, że odpowiedź jest kontrolowanym błędem dla klienta.
5. Potwierdź, że nie ma traceback, ścieżki pliku, nazwy funkcji, numeru linii, typu wyjątku, debug object ani wrażliwej wartości.
6. Wyślij `GET /cgi-bin/phpinfo.php`.
7. Potwierdź, że endpoint zwraca `404` albo `403`.
8. Potwierdź, że odpowiedź nie zawiera `phpinfo()`, `PHP Version`, `SECRET_KEY` ani zmiennych środowiskowych.
9. Potwierdź, że publiczny HTML nie zawiera debug links.
10. Potwierdź, że frontend nie renderuje raw backend errors.

## Automation idea

```javascript
test("invalid user ID does not expose debug information", async () => {
  const response = await request(app).get("/api/user/admin");

  expect(response.status).toBe(400);
  expect(response.body.error).toBe("Invalid user ID");

  const body = JSON.stringify(response.body);

  expect(body).not.toContain("Traceback");
  expect(body).not.toContain("/app/app.py");
  expect(body).not.toContain("ValueError");
  expect(body).not.toContain("debug_info");
  expect(body).not.toContain("flag");
  expect(body).not.toContain("SECRET");
  expect(body).not.toContain("TOKEN");
});
```

## Logging and alerting ideas

Dla powtarzających się invalid requests albo prób dostępu do ścieżek debug/config rozważ logowanie:

- endpointu,
- nazwy invalid parameter i bezpiecznego podsumowania,
- authenticated user ID, jeśli istnieje,
- timestamp,
- source IP lub request metadata,
- response status,
- reguły lub validation failure.

Nie loguj sekretów, session cookies ani wrażliwych tokenów.

Przyszły review A09 powinien sprawdzić, czy powtarzające się sondowanie ścieżek debug/config jest widoczne dla defenderów.

## Acceptance criteria

Poprawka jest akceptowalna tylko wtedy, gdy:

- produkcyjny debug mode jest wyłączony,
- invalid input zwraca generyczny client-facing error,
- publiczne odpowiedzi nie ujawniają stack trace'ów, wewnętrznych ścieżek, typów wyjątków ani sekretów,
- szczegółowe błędy są dostępne tylko w chronionych logach server-side,
- valid API behaviour nadal działa,
- frontend error handling pozostaje przyjazny dla użytkownika,
- testy pokrywają zarówno valid, jak i invalid cases,
- publiczne debug pages są usunięte albo zablokowane,
- ujawnione sekrety zostały obrócone,
- production artifacts nie zawierają znanych debug files,
- testy failują, jeśli verbose debug strings pojawią się ponownie.
