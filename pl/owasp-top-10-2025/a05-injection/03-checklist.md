# A05: Injection Checklist

Używaj tej checklisty podczas nauki, code review albo autoryzowanych testów.

## Ogólne pytania o injection

- Czy dane kontrolowane przez użytkownika trafiają do interpretera albo silnika przetwarzającego?
- Czy dane mogą zmienić strukturę zapytania, komendy, szablonu, wyrażenia albo instrukcji?
- Czy dane są traktowane jako dane, czy mogą stać się składnią?
- Jaka dokładnie struktura danych trafia do backendu?
- Czy jest to string, tablica, obiekt, zagnieżdżony obiekt, JSON, pole formularza, query parameter, cookie albo nagłówek?
- Czy walidacja jest używana jako jedyna linia obrony?
- Czy bezpieczna implementacja używa safe APIs, parametryzacji, ścisłych schematów, kontroli typów albo allowlistowanych operacji?

## Kontrole SQL Injection

- Czy zapytania SQL są budowane przez konkatenację stringów z danymi użytkownika?
- Czy parametry URL, body requestu, cookies albo nagłówki są używane w zapytaniach SQL?
- Czy dodanie pojedynczego apostrofu zmienia odpowiedź albo powoduje błąd?
- Czy dodanie komentarza SQL naprawia zepsute zapytanie?
- Czy warunki true/false zmieniają zachowanie odpowiedzi?
- Czy `UNION SELECT` może zwrócić dodatkowe dane w odpowiedzi?
- Czy blind SQLi można wykryć przez różnice w odpowiedziach albo timing?
- Czy błędy bazy danych są widoczne dla użytkownika?

## Kontrole NoSQL Injection

### Input i struktura danych

- Czy endpoint oczekuje prostego stringa, ale akceptuje obiekt albo tablicę?
- Czy JSON może zawierać zagnieżdżone obiekty tam, gdzie oczekiwany jest string?
- Czy notacja formularza URL-encoded typu `field[operator]=value` tworzy obiekt po stronie serwera?
- Czy body parser zachowuje klucze kontrolowane przez użytkownika zaczynające się od `$`?
- Czy użytkownik kontroluje nazwy pól, operatory, sortowanie, projekcję albo filtry?

### Operator Injection

- Czy input może stać się obiektem operatora, na przykład `$ne`, `$nin`, `$gt` albo `$regex`?
- Czy zmiana wartości w zagnieżdżony obiekt zmienia logikę logowania albo lookupu?
- Czy query może zwrócić użytkownika bez weryfikacji podanego hasła?
- Czy warunki regex tworzą oracle true/false do ekstrakcji sekretu?

### Syntax Injection

- Czy input użytkownika jest doklejany do `$where` albo innego custom JavaScript expression?
- Czy pojedynczy apostrof powoduje błąd albo zmianę zachowania?
- Czy kontrolowane warunki true i false dają różne odpowiedzi?
- Czy warunek zawsze prawdziwy zwraca rekordy spoza zamierzonego filtra?
- Czy widoczne są szczegółowe błędy MongoDB albo interpretera JavaScript?

### Odkrywanie attack surface

- Czy widoczny URL w przeglądarce jest tym endpointem, który naprawdę pobiera dane?
- Czy strona wykonuje background fetch/XHR requests?
- Czy podatny request widać dopiero w Burp HTTP history albo DevTools?
- Czy redirecty ukrywają endpoint, który faktycznie przetwarza dane?

## Dowody, których szukam

- komunikaty błędów bazy albo interpretera,
- HTTP 500 albo kontrolowany błąd po syntax-sensitive input,
- różne body odpowiedzi dla warunku true i false,
- lookup zwracający innego użytkownika,
- dodatkowe produkty albo rekordy w odpowiedzi,
- różnica długości odpowiedzi albo marker w treści,
- potwierdzona długość sekretu przez powtarzalne warunki,
- jeden poprawny znak dla każdej pozycji sekretu.

## Kontrole bezpiecznej implementacji

- Używane są prepared statements / parameterized queries.
- Dane użytkownika są przekazywane jako wartości, a nie doklejane do stringów SQL.
- Allowlisty są używane tam, gdzie wartości są ograniczone, na przykład kategorie albo pola sortowania.
- Konta bazy danych działają zgodnie z least privilege.
- Inputy NoSQL są walidowane przez ścisłe schematy.
- Pola uwierzytelniania są egzekwowane jako prymitywne stringi.
- Zagnieżdżone obiekty i input w kształcie operatorów są odrzucane tam, gdzie nie są wymagane.
- Dane użytkownika nie są wstawiane do `$where` ani custom JavaScript.
- Hasła są weryfikowane przez porównanie z hashem, a nie query z plaintext password.
- Produkcja nie ujawnia szczegółowych błędów bazy ani interpretera.
- Rate limiting i monitoring utrudniają automatyczną ekstrakcję.
- Testy regresji obejmują payloady stringowe i object-shaped payloads.

## Kąt frontend/AppSec

- Nie zakładaj, że wartości są bezpieczne, bo pochodzą z linków wygenerowanych przez frontend.
- Nie zakładaj, że hidden fields, cookies albo predefiniowane linki kategorii są zaufane.
- Nie traktuj GET vs POST jako mechanizmu bezpieczeństwa.
- Sprawdzaj background API requests, nie tylko pasek adresu.
- Granicą bezpieczeństwa jest sposób, w jaki backend parsuje input i buduje zapytanie.

Dłuższe checklisty tematyczne:

- [SQL Injection cheat sheet](sql-injection/cheat-sheet.md)
- [NoSQL Injection cheat sheet](nosql-injection/cheat-sheet.md)
