# A05: Injection - Pomysły na testy regresji

Testy regresji powinny potwierdzać, że wcześniej niebezpieczne dane wejściowe są teraz traktowane jako dane, odrzucane w kontrolowany sposób albo blokowane przed dotarciem do wykonywalnej składni zapytania.

## Testy regresji SQL Injection

- Pojedynczy apostrof w kategorii albo wyszukiwanej wartości nie powinien powodować błędu 500.
- Komentarze SQL, takie jak `--`, nie powinny zmieniać zachowania zapytania.
- Payloady boolean, takie jak `' AND '1'='1` i `' AND '1'='2`, nie powinny powodować znaczących różnic w odpowiedzi.
- `UNION SELECT NULL,NULL` nie powinien zwracać dodatkowych wierszy ani danych.
- `UNION SELECT 'abc','def'` nie powinien renderować danych kontrolowanych przez atakującego.
- Wartości cookie, takie jak `TrackingId`, nie powinny pozwalać na zmianę logiki zapytania SQL.
- Payloady SQLi powinny być obsługiwane bezpiecznie jako zwykłe stringi.

## Testy regresji NoSQL Injection

### Egzekwowanie typów prymitywnych

- `username` i `password` akceptują wyłącznie stringi.
- Tablice, obiekty i zagnieżdżone struktury są odrzucane kontrolowaną odpowiedzią `400`.
- JSON taki jak `{"username":{"$ne":null}}` jest odrzucany.
- Dane formularza używające bracket notation do tworzenia obiektów są odrzucane tam, gdzie oczekiwany jest string.
- Nieznane pola są odrzucane albo usuwane przez walidację schematu.

### Kontrola operatorów query

- Input klienta nie może wprowadzić `$ne`, `$nin`, `$gt`, `$regex`, `$where` ani innych operatorów query.
- Wartość zawierająca `$` pozostaje danymi albo jest odrzucana; nie staje się kluczem query.
- Użytkownik nie kontroluje nazw pól, projekcji, sortowania ani surowych filtrów, chyba że są jawnie allowlistowane.
- Input przypominający operator nie omija uwierzytelniania ani nie zwraca pierwszego dokumentu z bazy.

### Syntax Injection

- Apostrof w lookupie albo kategorii nie powoduje błędu MongoDB ani interpretera JavaScript.
- JavaScript-style boolean expressions nie zmieniają zestawu wyników.
- Warunki zawsze prawdziwe nie zwracają unreleased products, cudzych rekordów ani danych innego użytkownika.
- Żaden input użytkownika nie jest doklejany do `$where` ani custom JavaScript.

### Odporność na boolean oracle

- Payloady true i false nie powodują znaczącej różnicy zależnej od sekretu.
- Lookup użytkownika nie ujawnia, czy warunek dotyczący hasła albo sekretu jest prawdziwy.
- Error body, długość odpowiedzi i statusy są normalizowane tam, gdzie to potrzebne.
- Powtarzalne próbkowanie uruchamia rate limiting, alerting albo tymczasowe kontrole.

### Projekt uwierzytelniania

- Aplikacja pobiera użytkownika tylko po dokładnym, zwalidowanym username albo email.
- Porównanie hasła używa bezpiecznej funkcji weryfikacji hasha.
- Hasła nie są przechowywane ani odpytywane jako plaintext.
- Nieudane uwierzytelnianie nie zwraca innego rekordu użytkownika.
- Sukces logowania nie może wynikać wyłącznie z tego, że query zwróciło co najmniej jeden dokument.

## Oczekiwania wobec zachowania aplikacji

- Niepoprawne dane wejściowe powinny zwracać kontrolowaną odpowiedź, a nie błąd bazy danych.
- Znane kategorie powinny być wybierane przez allowlistowane identyfikatory albo bezpieczne parametry zapytania.
- Nieznane kategorie powinny zwracać brak wyników albo bezpieczny błąd walidacji.
- Aplikacja nie powinna ujawniać surowych błędów SQL, MongoDB, JavaScript, drivera ani stack trace.
- Background API endpoints powinny egzekwować taką samą walidację i autoryzację jak widoczne strony.

## Kontrole na poziomie kodu

- Zapytania powinny używać prepared statements / parameterized queries.
- Żadna wartość kontrolowana przez użytkownika nie powinna być doklejana bezpośrednio do stringów SQL.
- Filtry NoSQL powinny być budowane z pól i operatorów wybranych po stronie serwera.
- Dane requestu powinny być konwertowane do zwalidowanego modelu wewnętrznego przed budową query.
- Użycie ORM powinno unikać niebezpiecznego raw query construction.
- Konta bazy danych nie powinny mieć niepotrzebnych uprawnień.

## Checklista regresji dla security findingu

Dla każdej poprawki injection potwierdź, że:

- podatny parametr nie zmienia już zachowania interpretera,
- niebezpieczne dane są traktowane jako wartości albo odrzucane,
- różnice w odpowiedziach nie ujawniają już prawdy o sekretach,
- danych wrażliwych nie da się pobrać,
- przejęcie konta administratora nie jest już możliwe,
- istnieją testy zapobiegające powrotowi problemu.
