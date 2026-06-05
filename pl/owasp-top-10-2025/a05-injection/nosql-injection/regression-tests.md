# NoSQL Injection Regression Tests

Testy regresji powinny potwierdzać, że input użytkownika nie może już stać się operatorem, obiektem query ani wykonywalną składnią.

## Type enforcement

- `username` jako string działa zgodnie z oczekiwaniem.
- `username` jako obiekt jest odrzucany.
- `username` jako tablica jest odrzucany.
- `password` jako obiekt jest odrzucany.
- Nieznane pola są odrzucane albo ignorowane zgodnie ze schematem.

## Operator payloads

- `{"username":{"$ne":null}}` nie omija lookupu ani logowania.
- `{"password":{"$ne":null}}` nie powoduje sukcesu logowania.
- `$regex` nie może zostać wprowadzony przez klienta w polach logowania.
- `$where` nie może zostać wprowadzony przez klienta.
- Bracket notation typu `username[$ne]=test` nie tworzy operatora query.

## Syntax payloads

- Apostrof w parametrze lookup nie powoduje błędu interpretera.
- Warunek always-true nie zwraca innego użytkownika.
- Warunek always-false nie tworzy odróżnialnego sekretnego markera.
- JavaScript-like expressions są traktowane jako dane albo odrzucane.

## Oracle resistance

- Warunki dotyczące długości hasła nie zmieniają odpowiedzi.
- Warunki dotyczące pozycji znaków nie zmieniają odpowiedzi.
- Status, response body i response length nie ujawniają truth value sekretu.
- Powtarzalne próbkowanie uruchamia rate limiting albo alerting.

## Authentication

- Użytkownik jest pobierany po dokładnym, zwalidowanym username albo email.
- Hasło jest porównywane przez bezpieczną weryfikację hasha.
- Hasła nie są przechowywane ani odpytywane jako plaintext.
- Nieudane logowanie nie zwraca żadnego alternatywnego rekordu.

## Error handling

- Produkcja nie pokazuje błędów MongoDB.
- Produkcja nie pokazuje błędów JavaScript parsera.
- Produkcja nie pokazuje stack trace ani fragmentów query.
- Błędy walidacji są kontrolowane i spójne.

## Completion criteria

Poprawka jest wiarygodna, gdy:

- typy inputu są egzekwowane,
- struktura query jest kontrolowana przez backend,
- operatory nie mogą pochodzić od klienta,
- budowanie custom JavaScript query zostało usunięte albo zabezpieczone,
- sensitive fields nie są queryable,
- testy automatyczne obejmują string payloads i object-shaped payloads.
