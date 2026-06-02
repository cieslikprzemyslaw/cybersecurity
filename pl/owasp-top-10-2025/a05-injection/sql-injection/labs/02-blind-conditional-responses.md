# Lab: Blind SQL Injection with Conditional Responses

## Źródło

PortSwigger Web Security Academy: **Blind SQL injection with conditional responses**

## Cel

Użyć boolean-based blind SQL Injection, aby wyciągnąć hasło administratora i zalogować się.

## Podatna funkcja

Podatnym inputem było cookie `TrackingId`.

Kontrolowany input:

```text
Cookie: TrackingId=...
```

## Dlaczego to było blind SQLi

Aplikacja nie zwracała danych z bazy bezpośrednio w odpowiedzi.

Zamiast tego pokazywała różne zachowanie zależnie od tego, czy wstrzyknięty warunek SQL był prawdziwy czy fałszywy.

## Marker TRUE/FALSE

Markerem było:

```text
Welcome back
```

Zaobserwowane zachowanie:

```text
Normalny TrackingId -> Welcome back
Warunek TRUE -> Welcome back
Warunek FALSE -> brak Welcome back
```

To stworzyło oracle true/false.

## Flow testowy

### 1. Potwierdzenie zachowania warunkowego

Warunek true utrzymywał komunikat `Welcome back`.

Warunek false usuwał komunikat `Welcome back`.

To potwierdziło boolean-based blind SQLi.

### 2. Potwierdzenie tabeli users

Subquery sprawdziło, czy tabela `users` istnieje i może zwrócić wiersz.

Wynik:

```text
Welcome back = yes
```

### 3. Potwierdzenie użytkownika administrator

Drugie subquery sprawdziło, czy istnieje użytkownik `administrator`.

Wynik:

```text
Welcome back = yes
```

### 4. Ustalenie długości hasła

Warunek długości był testowany do momentu, gdy odpowiedź zwróciła `Welcome back`.

Wynik:

```text
długość hasła administratora = 20
```

### 5. Ekstrakcja hasła znak po znaku

Burp Intruder został użyty do automatyzacji sprawdzeń true/false dla każdej pozycji i każdego znaku.

Koncepcyjnie każdy request pytał:

```text
Czy znak N hasła administratora jest równy X?
```

Jeśli odpowiedź zawierała `Welcome back`, testowany znak był poprawny.

## Podsumowanie dowodów

```text
TrackingId normalny -> Welcome back
TrackingId + warunek TRUE -> Welcome back
TrackingId + warunek FALSE -> brak Welcome back
test tabeli users -> Welcome back
test użytkownika administrator -> Welcome back
długość hasła 20 -> Welcome back
sprawdzenia znak po znaku -> hasło odzyskane
```

## Realny wpływ

Podatność pozwalała wyciągnąć dane wrażliwe bez bezpośredniego wyświetlania wyników bazy. Hasło administratora zostało odzyskane przy użyciu odpowiedzi boolean, co doprowadziło do przejęcia konta.

## Błędy i proces nauki

- Blind SQLi było trudniejsze niż UNION-based SQLi, bo dane nie były wyświetlane bezpośrednio.
- Kluczowe było znalezienie wiarygodnego markera odpowiedzi.
- Proces wydawał się wolny, bo każdy znak wymagał sprawdzeń true/false.
- Burp Intruder pomógł zautomatyzować powtarzalne sprawdzenia.
- Najważniejsza lekcja: blind SQLi polega na zbudowaniu oracle, a nie na jednym magicznym payloadzie.

## Remediacja developerska

- Użyj prepared statements / parameterized queries.
- Traktuj wartości cookie jako niezaufany input.
- Unikaj unsafe raw SQL w logice sesji/tracking.
- Używaj bezpiecznych wzorców ORM.
- Używaj kont bazy danych z least privilege.
- Unikaj różnic w odpowiedzi, które ujawniają wyniki zapytań backendowych.
- Dodaj testy regresji dla payloadów true/false SQLi.

## Pomysły na testy regresji

- Wstrzyknięcie warunków true i false do `TrackingId` nie powinno zmieniać obecności `Welcome back`.
- Wartości cookie powinny być traktowane jako dane, a nie składnia SQL.
- Sprawdzanie długości hasła nie powinno być możliwe przez różnice w odpowiedziach.
- Ekstrakcja znak po znaku nie powinna być możliwa przez powtarzane requesty warunkowe.
