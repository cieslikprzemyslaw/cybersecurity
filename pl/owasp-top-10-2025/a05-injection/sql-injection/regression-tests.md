# SQL Injection Regression Tests

## Testy regresji UNION-based SQLi

Po naprawie UNION-based SQLi sprawdź, że:

- dodanie pojedynczego apostrofu nie powoduje błędu bazy ani odpowiedzi 500,
- dodanie `--` nie zmienia zachowania zapytania,
- `UNION SELECT NULL,NULL` jest traktowany jak string albo bezpiecznie odrzucany,
- `UNION SELECT 'abc','def'` nie renderuje wstrzykniętych wartości,
- próba wybrania danych z `users` nie ujawnia danych,
- nieznane kategorie zwracają bezpieczną odpowiedź.

## Testy regresji blind SQLi

Dla blind SQLi w cookies albo innych wartościach sprawdź, że:

- warunki TRUE i FALSE nie wpływają na widoczne markery,
- `Welcome back` albo podobne markery UI nie są kontrolowane przez wstrzykniętą logikę SQL,
- `TrackingId` albo równoważne cookie jest traktowane jako dane,
- sprawdzanie długości hasła nie jest możliwe przez różnice w odpowiedziach,
- ekstrakcja znak po znaku nie jest możliwa.

## Kontrole regresji w code review

- Brak konkatenacji stringów zapytań z niezaufanym inputem.
- Prepared statements są używane konsekwentnie.
- Raw queries w ORM są sprawdzone.
- Inputy z ograniczonym zestawem poprawnych wartości używają allowlist.
- Uprawnienia użytkownika bazy danych są ograniczone.
- Błędy SQL nie są ujawniane użytkownikom.
