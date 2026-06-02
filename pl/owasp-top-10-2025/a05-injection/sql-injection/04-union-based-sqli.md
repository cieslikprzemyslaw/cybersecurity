# UNION-Based SQL Injection

## Czym jest

UNION-based SQL Injection używa operatora SQL `UNION`, aby połączyć wynik oryginalnego zapytania z wynikiem wstrzykniętego zapytania.

Celem jest zwykle sprawienie, aby aplikacja wyświetliła dane z innej tabeli.

## Kluczowa zasada

`UNION SELECT` musi zwracać tę samą liczbę kolumn co oryginalne zapytanie.

Jeśli oryginalne zapytanie zwraca 2 kolumny, wstrzyknięte zapytanie też musi zwracać 2 kolumny:

```sql
UNION SELECT NULL,NULL
```

Jeśli liczba kolumn jest błędna, baza zwykle zwróci błąd.

## Dlaczego NULL jest przydatny

`NULL` jest przydatny, bo pasuje do wielu typów kolumn.

Pomaga ustalić liczbę kolumn bez znajomości dokładnych typów danych.

Ważne rozróżnienie:

```text
NULL,NULL potwierdza liczbę kolumn.
Nie dowodzi jeszcze użytecznej ekstrakcji danych.
```

## Testowanie kolumn tekstowych

Po ustaleniu liczby kolumn sprawdź, czy wartości tekstowe mogą zostać wyświetlone:

```sql
UNION SELECT 'abc','def'
```

Jeśli `abc` i `def` pojawiają się w odpowiedzi, dowodzi to, że wstrzyknięty wynik jest widoczny w HTML i że kolumny mogą przyjmować tekst.

## Flow dowodów z laba

W labie PortSwigger flow dowodów wyglądał tak:

```text
normalna wartość category -> 200 OK
pojedynczy apostrof -> błąd 500
pojedynczy apostrof + komentarz SQL -> 200 OK
UNION SELECT NULL -> błąd 500
UNION SELECT NULL,NULL -> 200 OK
UNION SELECT 'abc','def' -> abc i def pojawiły się w HTML
UNION SELECT username,password FROM users -> nazwy użytkowników i hasła pojawiły się w HTML
```

## Realny wpływ

UNION-based SQLi może pozwolić atakującemu pobrać dane wrażliwe z innych tabel, na przykład nazwy użytkowników i hasła.

W labie doprowadziło to do przejęcia konta administratora.

## Poprawka developerska

Użyj prepared statements / parameterized queries. Dla filtrów kategorii użyj także allowlisty wartości kategorii, jeśli to pasuje do logiki aplikacji.
