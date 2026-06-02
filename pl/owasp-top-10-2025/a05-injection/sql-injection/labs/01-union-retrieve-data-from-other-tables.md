# Lab: UNION-Based SQL Injection - Retrieving Data from Other Tables

## Źródło

PortSwigger Web Security Academy: **SQL injection UNION attack, retrieving data from other tables**

## Cel

Wykorzystać podatność UNION-based SQL Injection, aby pobrać dane z innej tabeli i zalogować się jako administrator.

## Podatna funkcja

Podatną funkcją był filtr kategorii produktów.

Kontrolowany input:

```text
/filter?category=Pets
```

Parametr `category` wyglądał na używany w backendowym zapytaniu SQL.

## Interpreter

Właściwym interpreterem była baza danych / silnik SQL.

## Flow testowy

### 1. Normalny request

Normalna wartość kategorii zwróciła `200 OK` i wyświetliła produkty.

### 2. Test apostrofu

Dodanie pojedynczego apostrofu spowodowało `500 Internal Server Error`.

To sugerowało, że apostrof zepsuł składnię SQL.

### 3. Test komentarza SQL

Dodanie komentarza SQL przywróciło normalne zachowanie strony.

To sugerowało, że apostrof zamknął string SQL, a komentarz usunął problematyczną resztę oryginalnego zapytania.

### 4. Liczba kolumn

Test z jednym `NULL` spowodował błąd.

Test z dwoma wartościami `NULL` zwrócił poprawną odpowiedź.

Wniosek:

```text
Oryginalne zapytanie prawdopodobnie zwracało 2 kolumny.
```

### 5. Kolumny tekstowe

Test z wartościami tekstowymi, takimi jak `abc` i `def`, zwrócił obie wartości w odpowiedzi HTML.

To potwierdziło:

- wynik UNION był widoczny,
- obie kolumny mogły obsłużyć tekst,
- wstrzyknięty wynik SELECT był renderowany na stronie.

### 6. Ekstrakcja danych

Następnym krokiem było pobranie nazw użytkowników i haseł z tabeli `users`.

Odpowiedź pokazała użytkowników, w tym:

```text
administrator
```

Hasło administratora zostało potem użyte do logowania i rozwiązania laba.

## Podsumowanie dowodów

```text
Normalna kategoria -> 200 OK
Pojedynczy apostrof -> błąd 500
Cudzysłów + komentarz -> 200 OK
UNION SELECT NULL -> błąd
UNION SELECT NULL,NULL -> 200 OK
UNION SELECT 'abc','def' -> wartości widoczne w HTML
UNION SELECT username,password FROM users -> dane logowania widoczne w HTML
```

## Realny wpływ

Problem pozwalał ujawnić dane wrażliwe z innej tabeli, w tym nazwy użytkowników i hasła. Doprowadziło to do przejęcia konta administratora.

## Błędy i proces nauki

- Na początku potrzebowałem przypomnienia, żeby użyć komentarza SQL po zepsuciu zapytania apostrofem.
- Nauczyłem się, że `NULL,NULL` pomaga ustalić liczbę kolumn, ale samo nie dowodzi ekstrakcji danych.
- Test `abc/def` był ważny, bo potwierdził, że wstrzyknięte wartości tekstowe są widoczne w odpowiedzi.
- Ten lab pokazał różnicę między kontrolowanym zbieraniem dowodów a losowym zgadywaniem payloadów.

## Remediacja developerska

- Użyj prepared statements / parameterized queries.
- Nie doklejaj wartości kategorii do stringów SQL.
- Użyj allowlisty dla znanych wartości kategorii.
- Nie ujawniaj szczegółowych błędów bazy danych.
- Używaj kont bazy danych z least privilege.
- Dodaj testy regresji dla payloadów SQLi.

## Pomysły na testy regresji

- Pojedynczy apostrof w `category` nie powinien powodować błędu 500.
- Komentarze SQL nie powinny wpływać na zachowanie zapytania.
- `UNION SELECT NULL,NULL` nie powinien zwracać dodatkowych wierszy.
- Wstrzyknięte wartości tekstowe nie powinny pojawiać się w odpowiedzi.
- Próby pobrania danych z `users` powinny kończyć się bezpiecznie.
