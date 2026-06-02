# Example Finding: Boolean-Based Blind SQL Injection in TrackingId Cookie

## Summary

Cookie `TrackingId` jest podatne na boolean-based blind SQL Injection. Atakujący może wnioskować informacje z bazy danych, obserwując, czy komunikat `Welcome back` pojawia się w odpowiedzi.

## Dotknięty parametr

```text
Cookie: TrackingId=...
```

## Dowody

Zaobserwowano następujące zachowanie:

```text
Normalny TrackingId -> Welcome back pojawia się
Warunek SQL TRUE -> Welcome back pojawia się
Warunek SQL FALSE -> Welcome back znika
```

To pokazuje, że wartość cookie wpływa na backendowe zapytanie SQL i że różnice w odpowiedzi mogą działać jako oracle true/false.

Dodatkowe sprawdzenia potwierdziły:

```text
tabela users istnieje
użytkownik administrator istnieje
długość hasła administratora = 20
hasło można wyciągnąć znak po znaku
```

## Impact

Atakujący może wyciągnąć dane wrażliwe, mimo że aplikacja nie wyświetla bezpośrednio wyników bazy. W scenariuszu labowym odzyskano hasło administratora i użyto go do dostępu do konta administratora.

## Przyczyna źródłowa

Aplikacja prawdopodobnie używa cookie `TrackingId` w zapytaniu SQL bez parametryzacji.

## Rekomendacja

- Użyj prepared statements / parameterized queries.
- Traktuj wartości cookie jako niezaufany input.
- Unikaj unsafe raw SQL w logice tracking/session.
- Używaj bezpiecznych wzorców ORM.
- Stosuj least privilege dla użytkowników bazy danych.
- Upewnij się, że zachowanie odpowiedzi nie ujawnia wartości logicznych zapytania.
- Dodaj testy regresji dla wzorców boolean-based SQLi.

## Testy regresji

- Warunki TRUE i FALSE w `TrackingId` nie powinny zmieniać markera odpowiedzi.
- Komunikat `Welcome back` nie powinien zależeć od wstrzykiwalnej logiki SQL.
- Wnioskowanie długości hasła nie powinno być możliwe.
- Ekstrakcja znak po znaku nie powinna być możliwa przez powtarzane requesty.
