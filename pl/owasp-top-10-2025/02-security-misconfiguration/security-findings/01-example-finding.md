# Przykładowe znalezisko: Verbose API error response ujawnia debug information i wrażliwe dane

## Summary

User Management API ujawnia verbose debug information, gdy nienumeryczny user ID zostanie podany do endpointu `/api/user/<user_id>`.

Odpowiedź zawiera wewnętrzne szczegóły implementacji, takie jak backendowa ścieżka pliku, nazwa funkcji, numer linii, typ wyjątku, traceback i wrażliwa wartość flagi labowej.

To zachowanie wskazuje na niebezpieczną produkcyjną obsługę błędów i mapuje się na OWASP Top 10 2025 A02: Security Misconfiguration.

## Affected area

- Funkcja: User Management API
- Endpoint: `GET /api/user/<user_id>`
- Dotknięte zachowanie: Obsługa błędu invalid user ID
- Typ użytkownika: Każdy użytkownik, który może dotrzeć do endpointu
- Mapowanie OWASP: A02:2025 Security Misconfiguration

## Severity

**Medium**

Problem ujawnia wrażliwe informacje techniczne i wrażliwą wartość labową. Sam w sobie nie daje bezpośrednio pełnego kompromisu systemu, ale ujawnione informacje mogą istotnie pomóc atakującemu w rozpoznaniu i kolejnych atakach.

Severity może wzrosnąć do High, jeśli ujawnione zostaną realne sekrety, credentials, tokeny lub exploitable internal paths.

## Risk / Impact

Użytkownik może wywołać verbose error output przez podanie invalid input.

Odpowiedź ujawnia techniczne szczegóły, które nie powinny być widoczne dla użytkowników, w tym:

- wewnętrzną ścieżkę pliku,
- nazwę funkcji backendowej,
- numer linii kodu źródłowego,
- typ wyjątku,
- traceback,
- wrażliwą wartość zawartą w exception message.

W prawdziwej aplikacji podobne zachowanie mogłoby ujawnić:

- klucze API,
- JWT secrets,
- database connection strings,
- zmienne środowiskowe,
- szczegóły frameworka,
- database errors,
- nazwy usług wewnętrznych,
- wewnętrzne ścieżki plików,
- strukturę kodu źródłowego.

Takie informacje mogą pomóc atakującemu zrozumieć implementację backendu i przygotować bardziej ukierunkowane ataki. Mogą też wspierać eksploitację innych podatności, takich jak path traversal, LFI, injection, SSRF, exposed debug endpoints lub insecure file access.

## Root cause

Aplikacja zwraca raw albo zbyt szczegółowe exception information do klienta, gdy input validation fails.

Prawdopodobne przyczyny:

- debug-style error handling włączony w kontekście production-like,
- brakujący lub niepełny global error handler,
- niebezpieczne konstruowanie exception messages,
- wrażliwe wartości zawarte w exception messages,
- brak generycznych client-facing error responses,
- input validation errors nie są obsługiwane bezpiecznie.

## Evidence

### Normalny request zwraca oczekiwaną odpowiedź

Request numeryczny zwraca generyczny obiekt użytkownika:

```http
GET /api/user/123
```

Przykładowa odpowiedź:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

### Invalid request ujawnia debug output

Request nienumeryczny wywołuje verbose debug output:

```http
GET /api/user/admin
```

Przykładowa odpowiedź z wrażliwymi wartościami zredagowanymi:

```json
{
  "debug_info": {
    "flag": "<redacted-lab-flag>",
    "error": "Invalid user ID format: admin. Flag: <redacted-lab-flag>",
    "traceback": "Traceback (most recent call last):\n  File \"/app/app.py\", line 21, in get_user\n    raise ValueError(...)\nValueError: Invalid user ID format: admin. Flag: <redacted-lab-flag>\n"
  }
}
```

Odpowiedź ujawnia:

- `/app/app.py`,
- `get_user`,
- `line 21`,
- `ValueError`,
- szczegóły traceback,
- wrażliwą wartość labową.

## Expected secure behaviour

Dla invalid user ID input API powinno zwracać kontrolowaną generyczną odpowiedź błędu.

Przykład:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid user ID"
}
```

Odpowiedź nie powinna zawierać:

- traceback,
- ścieżek plików,
- nazw funkcji,
- numerów linii,
- typów wyjątków,
- sekretów,
- flag,
- tokenów,
- zmiennych środowiskowych,
- raw backend error objects.

Szczegółowe informacje błędów powinny być logowane tylko server-side.

## Remediation

Wdróż production-safe error handling dla API.

Rekomendowane poprawki:

- Wyłącz debug mode na produkcji.
- Dodaj global API error handler.
- Waliduj `user_id` przed wejściem w logikę aplikacji.
- Zwracaj generyczne client-facing error messages dla invalid input.
- Loguj szczegółowe exception information tylko server-side.
- Usuń sekrety, flagi, tokeny i wartości środowiskowe z exception messages.
- Standaryzuj API error response formats.
- Zreviewuj inne endpointy pod podobne verbose error behaviour.
- Potwierdź separację konfiguracji production, staging i development.
- Dodaj automatyczne testy, które failują, jeśli verbose debug information pojawi się w odpowiedziach.

## Pomysły na testy regresji

Dodaj pokrycie regresyjne dla:

- nienumerycznych user IDs, takich jak `admin` i `abc`,
- malformed values, takich jak `%27`,
- bardzo długich wartości user ID,
- poprawnych numerycznych user IDs po poprawce,
- frontendowego renderowania błędów invalid-input,
- content checks, które failują, jeśli `Traceback`, `/app/app.py`, `ValueError`, `debug_info`, `flag`, `SECRET` lub `TOKEN` pojawią się w publicznych odpowiedziach.

Szczegółowe przypadki testowe są opisane w [04-regression-tests.md](../04-regression-tests.md).

## Mapowanie OWASP

- OWASP Top 10 2025: A02 Security Misconfiguration
- Powiązane przyszłe mapowanie: A10 Mishandling Exceptional Conditions
- Powiązany obszar review: production-safe error handling i debug information disclosure

## Developer takeaway

Invalid input jest normalny i oczekiwany. Bezpieczna aplikacja powinna obsługiwać go bezpiecznie.

Backend powinien zwracać jasny, ale generyczny client-facing error, a szczegółowe informacje debugowe powinny zostać w logach server-side.

Nie pozwalaj, żeby wewnętrzne szczegóły wyjątków stały się częścią publicznego kontraktu API.
