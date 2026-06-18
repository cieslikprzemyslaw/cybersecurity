# Ćwiczenie 01: API Error-Handling Review

## Scenariusz

Endpoint:

```http
POST /api/assessments/:assessmentId/evidence
```

Endpoint waliduje dane requestu, sprawdza, czy assessment istnieje, tworzy rekord evidence i używa globalnego error handlera dla nieoczekiwanych wyjątków.

## Co sprawdziłem

Oddzieliłem oczekiwane problemy requestu od nieoczekiwanych awarii po stronie serwera.

| Przypadek | Oczekiwana odpowiedź | Powód |
| --- | --- | --- |
| Brak wymaganych pól, np. `title` lub `content` | `400 Bad Request` | Request jest źle zbudowany albo niekompletny. |
| Nieobsługiwany typ evidence, np. `"admin"` | `400 Bad Request` | Wartość enum nie jest dozwolona przez kontrakt API. |
| Poprawnie wyglądający `assessmentId`, ale brak rekordu | `404 Not Found` | Zasób nie istnieje albo aplikacja nie chce go ujawniać. |
| Użytkownik nie jest zalogowany | `401 Unauthorized` | Klient nie potwierdził tożsamości. |
| Użytkownik jest zalogowany, ale nie ma dostępu | `403 Forbidden` albo celowo `404 Not Found` | Użytkownik nie może odczytać albo zmienić zasobu. |
| Repository, baza albo dependency rzuca nieoczekiwany błąd | kontrolowane `500 Internal Server Error` | To nieoczekiwany stan po stronie aplikacji albo zależności. |

## Co pomieszałem na początku

Na początku połączyłem brak `assessmentId` z autentykacją. Poprawiony model:

```text
zła ścieżka / brak segmentu w URL -> często 404 route not found
źle sformatowany parametr -> 400 Bad Request
brak logowania -> 401 Unauthorized
zalogowany, ale bez dostępu -> 403 Forbidden albo czasem 404 dla ukrycia zasobu
poprawnie wyglądające ID, ale brak rekordu DB -> 404 Not Found
nieoczekiwana awaria serwera/dependency -> kontrolowane 500
```

## Lekcja A10

Generyczne `500` chroni klienta przed internal details, ale nie dowodzi, że backend bezpiecznie odzyskał stan.

Odpowiedź dowodzi tylko tego, co dostał klient. Nie dowodzi:

- rollbacku bazy,
- cleanupu plików,
- zwolnienia zasobów,
- spójnego stanu,
- ani bezpiecznej obsługi side effects.

## Bezpieczne oczekiwane zachowanie

API powinno:

- odrzucać invalid input przed wejściem do repository/bazy,
- zwracać kontrolowane `4xx` dla expected validation i access failures,
- przekazywać unexpected exceptions do globalnego fallback error handlera,
- zwracać generyczną odpowiedź klienta z correlation/request ID,
- logować wystarczający kontekst wewnętrznie,
- nie ujawniać stack trace, SQL, filesystem paths, secrets ani internal hostnames.

## Testy regresji

Przydatne testy:

- brak `title` zwraca `400`,
- brak `content` zwraca `400`,
- nieobsługiwany evidence type zwraca `400`,
- brak assessmentu zwraca `404`,
- request bez logowania zwraca `401`,
- request bez uprawnień zwraca `403` albo celowe `404`,
- exception z repository zwraca kontrolowane `500`,
- `500` zawiera safe message i request ID,
- `500` nie ujawnia stack trace ani internal details.

## Frontend takeaway

Frontend validation poprawia UX, ale nie jest granicą bezpieczeństwa. Backend musi walidować request, egzekwować access control i zachować bezpieczny stan, gdy coś pójdzie nie tak.
