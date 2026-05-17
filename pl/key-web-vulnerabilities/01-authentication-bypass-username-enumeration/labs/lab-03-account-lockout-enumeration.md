# Lab 03 - Username Enumeration via Account Lock

Topic: Authentication Bypass / Username Enumeration  
Focus: Zachowanie account lockout jako sygnał username enumeration

## Goal

Celem tego laba było zrozumienie, jak account lockout, który normalnie jest zabezpieczeniem, może przypadkowo ujawnić, czy username istnieje.

## Input I Controlled

Kontrolowałem:

```text
username
password
liczbę powtarzanych prób logowania
kolejność requestów
```

Główne zachowanie testowe polegało na wysyłaniu kilku błędnych haseł dla tego samego username i porównywaniu reakcji aplikacji.

## What Happened

Dla losowych albo nieistniejących username’ów powtarzane błędne próby logowania nadal zwracały normalny generyczny komunikat błędu.

Dla jednego kandydata username powtarzane błędne próby wywołały komunikat o blokadzie konta. To pokazało, że aplikacja zliczała nieudane próby dla tego username, co wskazywało, że konto istnieje.

Po zidentyfikowaniu poprawnego username testowanie haseł pokazało jedną odpowiedź, która zachowywała się inaczej niż normalne błędne próby. Po zakończeniu czasu blokady te same dane pozwoliły na poprawne logowanie.

Ważne sygnały w response:

- normalne błędne próby zwracały `200 OK`,
- losowe username’y dalej zwracały generyczny błąd,
- jeden username wywołał komunikat o lockout po kilku próbach,
- jedna próba z hasłem zwróciła inny pattern odpowiedzi,
- poprawne logowanie zwróciło redirect do strony konta i nowe session cookie.

## What I Learned

Nauczyłem się, że mechanizmy bezpieczeństwa też mogą wyciekać informacje, jeśli są wdrożone niespójnie.

Account lockout jest przydatny, ale jeśli pojawia się tylko dla istniejących kont, atakujący może użyć go do username enumeration.

Nauczyłem się też, że ten typ problemu może nie być widoczny po jednym request. Trzeba testować zachowanie aplikacji przez kilka prób.

## Impact

W prawdziwej aplikacji atakujący mógłby użyć zachowania account lockout do identyfikacji poprawnych użytkowników.

Problem może wspierać:

- password spraying,
- credential stuffing,
- celowane próby brute-force,
- phishing skierowany do znanych użytkowników,
- nadużycie account lockout albo denial of service wobec prawdziwych użytkowników.

## Remediation Summary

Zobacz `../overview.md`, aby poznać ogólną sekcję remediation.

Lab-specific remediation:

- nie pokazywać komunikatów o lockout tylko dla istniejących username’ów,
- utrzymywać spójne odpowiedzi dla nieudanych prób logowania,
- stosować rate limiting spójnie dla poprawnych i błędnych username’ów,
- ostrożnie łączyć kontrole oparte o konto, IP, urządzenie i zachowanie użytkownika,
- unikać sytuacji, w której atakujący może łatwo blokować konta innym użytkownikom,
- logować i monitorować powtarzane nieudane próby logowania,
- alertować przy wzorcach password spraying i credential stuffing,
- rozważyć MFA dla wrażliwych kont.

## Main Takeaway

Account lockout jest przydatną ochroną, ale jeśli zachowuje się inaczej dla istniejących i nieistniejących użytkowników, staje się sygnałem username enumeration.
