# A05: Injection - Pomysły na testy regresji

Testy regresji powinny potwierdzać, że wcześniej niebezpieczne dane wejściowe są teraz traktowane jako dane, a nie wykonywalna składnia.

## Testy regresji SQL Injection

- Pojedynczy apostrof w kategorii albo wyszukiwanej wartości nie powinien powodować błędu 500.
- Komentarze SQL, takie jak `--`, nie powinny zmieniać zachowania zapytania.
- Payloady boolean, takie jak `' AND '1'='1` i `' AND '1'='2`, nie powinny powodować znaczących różnic w odpowiedzi.
- `UNION SELECT NULL,NULL` nie powinien zwracać dodatkowych wierszy ani danych.
- `UNION SELECT 'abc','def'` nie powinien renderować danych kontrolowanych przez atakującego.
- Wartości cookie, takie jak `TrackingId`, nie powinny pozwalać na zmianę logiki zapytania SQL.
- Payloady SQLi powinny być obsługiwane bezpiecznie jako zwykłe stringi.

## Oczekiwania wobec zachowania aplikacji

- Niepoprawne dane wejściowe powinny zwracać kontrolowaną odpowiedź, a nie błąd bazy danych.
- Znane kategorie powinny być wybierane przez allowlistowane identyfikatory albo bezpieczne parametry zapytania.
- Nieznane kategorie powinny zwracać brak wyników albo bezpieczny błąd walidacji.
- Aplikacja nie powinna ujawniać surowych komunikatów błędów bazy danych.

## Kontrole na poziomie kodu

- Zapytania powinny używać prepared statements / parameterized queries.
- Żadna wartość kontrolowana przez użytkownika nie powinna być doklejana bezpośrednio do stringów SQL.
- Użycie ORM powinno unikać niebezpiecznego raw query construction.
- Konta bazy danych nie powinny mieć niepotrzebnych uprawnień.

## Checklista regresji dla security findingu

Dla każdej poprawki SQLi potwierdź, że:

- podatny parametr nie zmienia już zachowania SQL,
- payloady są traktowane jako wartości,
- danych wrażliwych nie da się pobrać,
- przejęcie konta administratora nie jest już możliwe,
- istnieją testy zapobiegające powrotowi problemu.
