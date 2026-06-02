# Blind SQL Injection

## Czym jest

Blind SQL Injection występuje wtedy, gdy aplikacja jest podatna na SQL Injection, ale dane z bazy nie są zwracane bezpośrednio w odpowiedzi.

Zamiast tego atakujący wnioskuje informacje z różnic w odpowiedziach albo z czasu odpowiedzi.

## Boolean-based blind SQLi

Boolean-based blind SQLi używa warunków true i false.

Model mentalny:

```text
warunek true  -> odpowiedź zawiera marker
warunek false -> odpowiedź nie zawiera markera
```

W labie PortSwigger markerem było:

```text
Welcome back
```

To działało jako oracle true/false:

```text
Welcome back = TRUE
Brak Welcome back = FALSE
```

## Dlaczego jest trudniejsze niż UNION SQLi

W UNION-based SQLi dane mogą pojawić się bezpośrednio w odpowiedzi.

W blind SQLi atakujący nie widzi danych bezpośrednio. Zamiast tego zadaje bazie wiele pytań tak/nie.

Przykłady:

```text
Czy hasło administratora ma długość 20?
Czy pierwszy znak hasła to "a"?
Czy drugi znak hasła to "b"?
```

## Flow laba

Lab PortSwigger używał cookie `TrackingId`.

Flow dowodów:

```text
Normalny TrackingId -> Welcome back
Warunek TRUE -> Welcome back
Warunek FALSE -> brak Welcome back
tabela users istnieje -> Welcome back
użytkownik administrator istnieje -> Welcome back
długość hasła = 20 -> Welcome back
znaki wyciągnięte jeden po drugim z użyciem Intrudera
```

## Ważna lekcja

Blind SQLi nie polega na jednym magicznym payloadzie. Polega na zbudowaniu wiarygodnej wyroczni i ostrożnym jej użyciu.

## Impact

Boolean-based blind SQLi nadal może prowadzić do ujawnienia danych wrażliwych i przejęcia konta, nawet gdy aplikacja nie wyświetla bezpośrednio błędów bazy ani wyników zapytania.

## Poprawka developerska

- Użyj prepared statements / parameterized queries.
- Unikaj niebezpiecznego budowania raw SQL.
- Używaj bezpiecznych wzorców ORM.
- Stosuj least privilege dla kont bazy danych.
- Dodaj testy regresji dla payloadów true/false.
