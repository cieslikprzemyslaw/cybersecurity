# Lab 01 - Username Enumeration via Different Responses

Topic: Authentication Bypass / Username Enumeration  
Focus: Oczywista różnica w komunikatach błędów logowania

## Goal

Celem tego laba było zrozumienie, jak różne komunikaty błędów logowania mogą ujawnić, czy username istnieje.

## Input I Controlled

Kontrolowałem body requestu logowania:

```text
username
password
```

Request był wysyłany do endpointu logowania metodą POST jako dane form-encoded.

## What Happened

Próba logowania losowym username i losowym password zwróciła komunikat, że username jest niepoprawny.

Podczas testowania różnych username’ów z tym samym błędnym hasłem jeden username zwrócił inny komunikat, wskazujący na niepoprawne hasło.

Ta różnica pokazała, że aplikacja traktowała jeden username jako poprawny.

Ważne sygnały w response:

- HTTP status nadal był `200 OK` dla błędnych prób,
- response body zawierało różne komunikaty błędów,
- długość odpowiedzi lekko się zmieniła,
- późniejsze poprawne logowanie zwróciło `302` redirect do strony konta,
- poprawna odpowiedź ustawiła również nowe session cookie.

## What I Learned

Nauczyłem się, że username enumeration może być bardzo proste, gdy aplikacja pokazuje różne komunikaty dla błędnego username i błędnego hasła.

Najważniejsza lekcja to dokładne porównywanie odpowiedzi i zmienianie tylko jednego parametru naraz.

Najpierw:

```text
username = zmienny
password = stałe błędne hasło
```

Potem, po znalezieniu poprawnego username:

```text
username = znany kandydat
password = zmienny
```

## Impact

W prawdziwej aplikacji taki problem może pomóc atakującym zbudować listę poprawnych username’ów. Następnie mogą użyć tej listy do password spraying, credential stuffing, phishingu albo celowanych prób brute-force.

Problem nie ujawnia hasła bezpośrednio, ale zmniejsza pracę atakującego.

## Remediation Summary

Zobacz `../overview.md`, aby poznać ogólną sekcję remediation.

Lab-specific remediation:

- używać jednego generycznego komunikatu dla wszystkich nieudanych prób logowania,
- unikać komunikatów typu `Invalid username` albo `Incorrect password`,
- utrzymywać spójne status code dla błędnych logowań,
- unikać różnic w długości odpowiedzi, jeśli to możliwe,
- testować odpowiedzi logowania z poprawnymi i błędnymi username’ami,
- dodać rate limiting i monitoring powtarzanych nieudanych prób.

## Main Takeaway

Komunikaty błędów logowania nie mogą zdradzać, czy błędną częścią był username czy password.
