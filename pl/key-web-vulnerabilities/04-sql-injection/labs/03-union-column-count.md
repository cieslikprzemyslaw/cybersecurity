# SQL Injection UNION Attack - Ustalanie liczby kolumn

> Platforma: PortSwigger Web Security Academy  
> Lab: SQL injection UNION attack, determining the number of columns returned by the query  
> Status: Ukończony  
> Cel: Prywatne notatki AppSec, nie pełny walkthrough.

## TL;DR

Filtr kategorii był podatny na SQL Injection. Aby wykonać atak typu `UNION SELECT`, najpierw trzeba było ustalić, ile kolumn zwraca oryginalne zapytanie. Poprawna liczba została potwierdzona wtedy, gdy aplikacja zaakceptowała `UNION SELECT` z trzema wartościami `NULL` i zwróciła `HTTP/2 200 OK`.

## Co ćwiczyłem

Ćwiczyłem pierwszy krok w UNION-based SQL Injection: ustalenie liczby kolumn zwracanych przez oryginalne zapytanie.

To jest ważne, ponieważ oba zapytania łączone przez `UNION` muszą zwracać taką samą liczbę kolumn.

## Input, który kontrolowałem

Kontrolowanym inputem był parametr query `category`:

```http
GET /filter?category=Gifts HTTP/2
```

To był lepszy punkt wejścia niż endpoint szczegółów produktu.

## Pierwsza próba: zły parametr

Najpierw testowałem parametr `productId`:

```http
GET /product?productId=1 ... HTTP/2
```

Aplikacja zwróciła:

```text
Invalid product ID
```

To sugerowało, że `productId` prawdopodobnie nie był właściwym punktem wejścia dla tego labu albo był walidowany jako liczba, zanim trafił do zapytania.

Lepszym miejscem do testów był filtr kategorii produktów.

## Przejście do filtra kategorii

Następnie testowałem filtr kategorii:

```http
GET /filter?category=Gifts HTTP/2
```

To wyglądało jak lepszy cel, ponieważ zapytanie zwracało listę produktów, a wynik był renderowany na stronie.

## Pierwsze próby UNION

Testowałem `UNION SELECT` z wartościami liczbowymi i różną liczbą kolumn.

Przykładowo, większe testy liczbowe zwracały:

```http
HTTP/2 500 Internal Server Error
```

To sugerowało, że `UNION SELECT` nie pasował do struktury oryginalnego zapytania albo wybrane wartości nie były zgodne z oczekiwanymi typami kolumn.

## Ważna lekcja: liczba kolumn i typy danych

Aby `UNION SELECT` zadziałał, wstrzyknięte zapytanie musi pasować do oryginalnego zapytania pod dwoma względami:

1. ta sama liczba kolumn;
2. kompatybilne typy danych.

Używanie wartości liczbowych typu `1,2,3` może czasem powodować problemy z typami danych.

`NULL` jest przydatne przy testowaniu liczby kolumn, ponieważ jest kompatybilne z wieloma typami danych.

## Działający test liczby kolumn

Zaakceptowany test wyglądał tak:

```http
GET /filter?category=Gifts' UNION SELECT NULL,NULL,NULL-- - HTTP/2
```

Aplikacja zwróciła:

```http
HTTP/2 200 OK
```

To potwierdziło, że oryginalne zapytanie zwraca trzy kolumny.

## Dlaczego response wyglądał prawie normalnie

Response nie pokazał wyraźnych widocznych wartości, ponieważ wstrzyknięte wartości były `NULL`.

W HTML pojawił się jednak dodatkowy pusty wiersz tabeli:

```html
<tr>
</tr>
```

To sugerowało, że wiersz z `UNION SELECT` został zaakceptowany i dodany do result set, ale nie wyrenderował żadnego widocznego tekstu.

## Dlaczego to zadziałało

Oryginalne zapytanie zwracało trzy kolumny. Wstrzyknięty `UNION SELECT` również zwracał trzy kolumny:

```sql
UNION SELECT NULL,NULL,NULL
```

Ponieważ liczba kolumn się zgadzała, a `NULL` był kompatybilny typowo, baza zaakceptowała zapytanie.

## Wpływ

Ustalenie liczby kolumn jest zwykle krokiem przygotowawczym do bardziej zaawansowanego UNION-based SQL Injection.

Gdy liczba kolumn jest znana, atakujący może próbować ustalić, które kolumny mogą wyświetlać tekst, a następnie próbować pobierać dane z innych tabel.

## Remediacja dla developera

Aplikacja powinna zapobiegać SQL Injection przez użycie parameterized queries / prepared statements.

Zalecane poprawki:

- używać prepared statements;
- unikać konkatenacji SQL;
- walidować wartości kategorii / filtrów względem oczekiwanych wartości;
- stosować zasadę least privilege dla konta bazy danych;
- nie pokazywać błędów bazy danych użytkownikom;
- dodać testy bezpieczeństwa dla modyfikowanych parametrów query.

## Checklist do review

- Czy parametr filtra może wyjść z oryginalnego stringa SQL?
- Czy aplikacja zwraca błędy bazy danych po zmianie składni SQL?
- Czy `UNION SELECT` może zostać zaakceptowany?
- Czy liczba kolumn oryginalnego zapytania może zostać odkryta?
- Czy błędy bazy danych są ukryte przed zwykłymi użytkownikami?
- Czy używane są prepared statements?

## Najważniejsza lekcja

Przed pobieraniem danych przez `UNION SELECT` atakujący musi wiedzieć, ile kolumn zwraca oryginalne zapytanie. `NULL` jest przydatne przy testowaniu liczby kolumn, ponieważ jest kompatybilne z wieloma typami danych.
