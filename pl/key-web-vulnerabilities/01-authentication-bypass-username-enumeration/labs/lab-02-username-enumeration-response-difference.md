# Lab 02 - Username Enumeration via Subtly Different Responses

Topic: Authentication Bypass / Username Enumeration  
Focus: Małe różnice w prawie generycznych odpowiedziach logowania

## Goal

Celem tego laba było zrozumienie, że username enumeration nie zawsze opiera się na oczywistych komunikatach. Nawet mała różnica w response body może ujawnić poprawny username.

## Input I Controlled

Kontrolowałem body requestu logowania:

```text
username
password
```

Request był wysyłany do endpointu logowania metodą POST jako dane form-encoded.

## What Happened

Aplikacja wyświetlała komunikat błędu, który wyglądał generycznie dla nieudanych prób logowania.

Na początku wyglądało to bezpieczniej niż w poprzednim labie, ponieważ komunikat nie mówił jasno, czy problemem był username czy password.

Jednak podczas porównywania wielu odpowiedzi jedna odpowiedź zawierała bardzo małą różnicę w komunikacie ostrzegawczym, na przykład interpunkcję albo spację. To pokazało, że backend przechodził inną ścieżką logiki dla jednego username.

Przydatne sygnały w response:

- HTTP status pozostał `200 OK` dla błędnych prób,
- widoczny komunikat wyglądał prawie tak samo,
- surowy HTML zawierał małą różnicę,
- Burp Grep - Extract pomógł wyciągnąć sam komunikat ostrzegawczy,
- poprawne logowanie później zwróciło `302` redirect i nowe session cookie.

## What I Learned

Nauczyłem się, że generyczne komunikaty nie wystarczą, jeśli surowe odpowiedzi nie są naprawdę identyczne.

Małe różnice mają znaczenie, w tym:

- interpunkcja,
- końcowe spacje,
- długość odpowiedzi,
- ukryte różnice w HTML-u,
- inne branche template’u.

Nauczyłem się też, że Burp Grep - Extract jest przydatny do szybkiego porównywania wielu odpowiedzi. Zamiast ręcznie czytać cały HTML, wyciągnąłem tylko komunikat ostrzegawczy i porównałem tę kolumnę w Intruderze.

## Impact

W prawdziwej aplikacji atakujący może automatycznie porównywać odpowiedzi i wykrywać małe różnice, których zwykły użytkownik nie zauważy.

To nadal może pozwolić na identyfikację poprawnych username’ów i skupienie dalszych ataków na prawdziwych kontach.

## Remediation Summary

Zobacz `../overview.md`, aby poznać ogólną sekcję remediation.

Lab-specific remediation:

- używać jednej wspólnej stałej albo template’u dla komunikatu błędu,
- upewnić się, że wszystkie nieudane ścieżki logowania zwracają ten sam widoczny komunikat,
- unikać przypadkowych różnic w interpunkcji albo spacjach,
- testować surowe odpowiedzi HTML, nie tylko widok w przeglądarce,
- porównywać status code, długość odpowiedzi, redirecty, cookies i timing,
- dodać negatywne testy authentication do QA/security testing.

## Main Takeaway

Komunikat może wyglądać generycznie w przeglądarce, ale nadal wyciekać informacje przez małe różnice w surowej odpowiedzi.
