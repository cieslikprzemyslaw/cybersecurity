# Example Finding: UNION-Based SQL Injection in Category Filter

## Summary

Filtr kategorii produktów jest podatny na UNION-based SQL Injection. Atakujący może zmanipulować parametr `category`, aby pobrać dane z innej tabeli bazy danych.

## Dotknięty parametr

```text
GET /filter?category=...
```

Dotknięty input:

```text
category
```

## Dowody

Poniższa sekwencja testowa pokazała problem:

```text
Normalna wartość kategorii -> 200 OK
Pojedynczy apostrof -> 500 Internal Server Error
Pojedynczy apostrof + komentarz SQL -> normalna odpowiedź
UNION SELECT NULL -> błąd
UNION SELECT NULL,NULL -> poprawna odpowiedź
UNION SELECT wartości tekstowe -> wstrzyknięte wartości pojawiły się w HTML
UNION SELECT username,password FROM users -> dane logowania użytkowników pojawiły się w HTML
```

To dowodzi, że parametr `category` może zmienić zapytanie SQL i że dane z innej tabeli mogą zostać wyrenderowane w odpowiedzi.

## Impact

Atakujący może pobrać informacje wrażliwe z bazy danych. W scenariuszu labowym obejmowało to nazwy użytkowników i hasła z tabeli `users`, co pozwoliło przejąć konto administratora.

## Przyczyna źródłowa

Aplikacja prawdopodobnie buduje zapytanie SQL przez doklejanie parametru `category` do stringa SQL.

## Rekomendacja

- Użyj prepared statements / parameterized queries.
- Nie doklejaj wartości kontrolowanych przez użytkownika do stringów SQL.
- Użyj allowlisty dla poprawnych wartości kategorii.
- Nie ujawniaj błędów bazy danych.
- Używaj kont bazy danych z least privilege.
- Dodaj testy regresji dla payloadów SQLi.

## Testy regresji

- Cudzysłów w wartości kategorii nie powinien powodować błędu bazy.
- Komentarze SQL nie powinny zmieniać zachowania zapytania.
- Payloady UNION SELECT nie powinny zwracać dodatkowych wierszy.
- Próby pobrania danych z tabeli `users` powinny kończyć się bezpiecznie.
