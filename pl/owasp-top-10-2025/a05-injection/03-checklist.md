# A05: Injection Checklist

Używaj tej checklisty podczas nauki, code review albo autoryzowanych testów.

## Ogólne pytania o injection

- Czy dane kontrolowane przez użytkownika trafiają do interpretera albo silnika przetwarzającego?
- Czy dane mogą zmienić strukturę zapytania, komendy, szablonu, wyrażenia albo instrukcji?
- Czy dane są traktowane jako dane, czy mogą stać się składnią?
- Czy walidacja jest używana jako jedyna linia obrony?
- Czy bezpieczna implementacja używa safe APIs, parametryzacji albo allowlistowanych operacji?

## Kontrole SQL Injection

- Czy zapytania SQL są budowane przez konkatenację stringów z danymi użytkownika?
- Czy parametry URL, body requestu, cookies albo nagłówki są używane w zapytaniach SQL?
- Czy dodanie pojedynczego apostrofu zmienia odpowiedź albo powoduje błąd?
- Czy dodanie komentarza SQL naprawia zepsute zapytanie?
- Czy warunki true/false zmieniają zachowanie odpowiedzi?
- Czy `UNION SELECT` może zwrócić dodatkowe dane w odpowiedzi?
- Czy blind SQLi można wykryć przez różnice w odpowiedziach albo timing?
- Czy błędy bazy danych są widoczne dla użytkownika?

## Dowody, których szukam

- komunikaty błędów SQL,
- HTTP 500 po wstrzyknięciu apostrofu,
- powrót odpowiedzi do normy po komentarzu SQL,
- różne odpowiedzi dla warunku true i false,
- dane z innej tabeli widoczne w odpowiedzi,
- zewnętrzny callback albo utworzenie pliku w scenariuszach OOB.

## Kontrole bezpiecznej implementacji

- Używane są prepared statements / parameterized queries.
- Dane użytkownika są przekazywane jako wartości, a nie doklejane do stringów SQL.
- Allowlisty są używane tam, gdzie wartości są ograniczone, na przykład kategorie albo pola sortowania.
- Konta bazy danych działają zgodnie z least privilege.
- Produkcja nie ujawnia szczegółowych błędów SQL.
- Testy regresji obejmują payloady w stylu SQLi.

## Kąt frontend/AppSec

- Nie zakładaj, że wartości są bezpieczne, bo pochodzą z linków wygenerowanych przez frontend.
- Nie zakładaj, że hidden fields, cookies albo predefiniowane linki kategorii są zaufane.
- Nie traktuj GET vs POST jako mechanizmu bezpieczeństwa.
- Granicą bezpieczeństwa jest sposób, w jaki backend buduje i wykonuje zapytanie SQL.

Dłuższa checklista SQLi znajduje się w [sql-injection/cheat-sheet.md](sql-injection/cheat-sheet.md).
