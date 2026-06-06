# A05: Injection - Pomysły na testy regresji

Testy regresji powinny potwierdzać, że wcześniej niebezpieczne dane wejściowe są teraz traktowane jako dane, odrzucane w kontrolowany sposób albo blokowane przed dotarciem do wykonywalnej składni zapytania lub komendy.

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

## Testy regresji OS Command Injection

### Kontrole architektury

- Funkcja używa biblioteki, SDK albo service API zamiast komendy systemowej, jeśli to możliwe.
- Żadna wartość kontrolowana przez użytkownika nie trafia do `sh -c`, `bash -c`, `cmd /c`, stringów PowerShella ani równoważnego interpretera shella.
- Executable jest wybierany po stronie serwera i nie jest kontrolowany przez klienta.
- Komenda i argumenty są przekazywane jako osobne wartości.
- API procesu jest skonfigurowane tak, aby nie używać shella.

### Input i argumenty

- Metaznaki shella są odrzucane przez ścisłą allowlistę albo pozostają literalnymi danymi bez zmiany wykonania.
- Separatory `;`, `&`, `&&`, `|` i `||` nie uruchamiają dodatkowych komend.
- Nowe linie i command substitution nie zmieniają wykonania procesu.
- Cudzysłowy nie pozwalają wyjść z kontekstu argumentu.
- Wartości zaczynające się od `-` nie mogą stać się nieoczekiwanymi opcjami komendy.
- `--` kończące parsowanie opcji jest używane tam, gdzie program to wspiera i gdzie ma to sens.
- Nazwy komend i wymagane flagi są hardcoded.
- Ograniczone wybory użytkownika są mapowane ze stabilnych identyfikatorów aplikacji na zatwierdzone wartości po stronie serwera.

### Zachowanie i side effects

- Normalny request i request z markerem timingowym mają porównywalne czasy odpowiedzi w ustalonej tolerancji.
- Zwiększenie wartości opóźnienia w inputcie nie powoduje proporcjonalnego wzrostu czasu odpowiedzi.
- Testowy input nie tworzy nieoczekiwanych plików.
- Testowy input nie zmienia zawartości plików, stanu aplikacji ani zachowania child process.
- Testowy input nie wywołuje outbound DNS, HTTP ani innych interakcji sieciowych.
- Output komendy nie może pojawić się w odpowiedzi HTTP, logach, plikach ani błędach przez unsafe execution.
- Niepoprawny input zwraca kontrolowaną odpowiedź, a nie surowy błąd procesu albo stack trace.

### Uprawnienia i ograniczenia

- Proces aplikacji działa jako dedykowane konto o niskich uprawnieniach.
- Proces nie może zapisywać do web root, jeśli funkcja jawnie tego nie wymaga.
- Proces nie może czytać niezwiązanych sekretów, plików konfiguracyjnych ani danych użytkowników.
- Proces nie może wykonywać niepotrzebnych binariów.
- Child processes dziedziczą tylko wymagane zmienne środowiskowe i dostęp do filesystemu.
- Process execution ma timeouty, limity outputu i limity zasobów.

### Code review i automatyzacja

- Static analysis wykrywa niebezpieczne użycia `exec`, `system`, `shell_exec`, `passthru`, `child_process.exec`, `shell=True`, `sh -c`, `cmd /c` i podobnych wzorców.
- Testy failują, jeśli refactor przywróci konkatenację stringów przy process-execution sink.
- Testy sprawdzają, że używana jest bezpieczna tablica argumentów, a nie jeden string komendy.
- Testy obejmują wartości oczekiwane, boundary values, wartości wyglądające jak opcje, whitespace, encoding i metaznaki.
- Logowanie zapisuje odrzucone albo nieudane próby wykonania bez zapisywania sekretów.

## Oczekiwania wobec zachowania aplikacji

- Niepoprawne dane wejściowe powinny zwracać kontrolowaną odpowiedź, a nie błąd bazy danych.
- Znane kategorie powinny być wybierane przez allowlistowane identyfikatory albo bezpieczne parametry zapytania.
- Nieznane kategorie powinny zwracać brak wyników albo bezpieczny błąd walidacji.
- Aplikacja nie powinna ujawniać surowych błędów SQL, MongoDB, JavaScript, drivera, shella, procesu ani stack trace.
- Background API endpoints powinny egzekwować taką samą walidację i autoryzację jak widoczne strony.
- Odpowiedź `500` powinna być zbadana, ale nie może być jedynym dowodem sukcesu albo porażki testu bezpieczeństwa.

## Kontrole na poziomie kodu

- Zapytania powinny używać prepared statements / parameterized queries.
- Żadna wartość kontrolowana przez użytkownika nie powinna być doklejana bezpośrednio do stringów SQL.
- Filtry NoSQL powinny być budowane z pól i operatorów wybranych po stronie serwera.
- Dane requestu powinny być konwertowane do zwalidowanego modelu wewnętrznego przed budową query.
- Użycie ORM powinno unikać niebezpiecznego raw query construction.
- Process execution powinno unikać shella i używać stałego executable z osobną listą argumentów.
- Konta bazy danych i konta systemowe nie powinny mieć niepotrzebnych uprawnień.

## Checklista regresji dla security findingu

Dla każdej poprawki injection potwierdź, że:

- podatny parametr nie zmienia już zachowania interpretera,
- niebezpieczne dane są traktowane jako wartości albo odrzucane,
- różnice w odpowiedziach nie ujawniają już prawdy o sekretach,
- timing i side effects nie dowodzą już ukrytego wykonania,
- danych wrażliwych ani plików nie da się pobrać,
- nieoczekiwanych komend ani child processes nie da się uruchomić,
- przejęcie konta administratora albo serwera nie jest już możliwe przez pierwotną ścieżkę,
- istnieją testy zapobiegające powrotowi problemu.

Rozszerzony zestaw testów dla OS Command Injection: [os-command-injection/regression-tests.md](os-command-injection/regression-tests.md).
