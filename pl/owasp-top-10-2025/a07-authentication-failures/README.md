# A07: Authentication Failures

Ten katalog zawiera moje praktyczne notatki do OWASP Top 10 2025 - A07: Authentication Failures.

## Status

**PASS — Frontend Developer przechodzący do AppSec**

Ukończona praca:

- TryHackMe: OWASP Top 10 2025 - wprowadzenie IAAA oraz sekcja A07 Authentication Failures,
- PortSwigger: Password reset broken logic,
- PortSwigger: 2FA simple bypass,
- krótkie review uwierzytelniania, odzyskiwania konta, cyklu życia sesji, enumeracji kont, rate limitingu i logoutu.

Status nie oznacza eksperckiej wiedzy IAM. Oznacza, że potrafię przeanalizować typowe flow uwierzytelniania, wskazać brakującą weryfikację po stronie serwera, oddzielić fakty od założeń oraz opisać użyteczne znalezisko.

## Zacznij tutaj

- [Overview](01-overview.md)
- [Laby i praktyka](02-labs-or-practice.md)
- [Checklista review uwierzytelniania](03-checklist.md)
- [Pomysły na testy regresyjne](04-regression-tests.md)
- [Learning notes](05-learning-notes.md)
- [Szczegółowe notatki z labów](labs-and-practice/README.md)
- [Przykładowe security findings](security-findings/)

## Główny model mentalny

```text
deklaracja tożsamości
    -> dowód uwierzytelnienia
    -> weryfikacja po stronie serwera
    -> sesja uwierzytelniona lub oczekująca na MFA
    -> cykl życia i unieważnienie sesji
```

Najważniejsze pytania review:

1. Jaka tożsamość jest deklarowana?
2. Jaki dowód ma ją potwierdzić?
3. Który komponent backendu weryfikuje dowód?
4. Czy dowód jest powiązany z właściwym kontem, akcją i próbą uwierzytelnienia?
5. Czy krok można pominąć, powtórzyć, przestawić albo zmodyfikować?
6. Kiedy sesja jest tworzona lub rotowana?
7. Jak sesja jest unieważniana?
8. Czy recovery albo fallback omija zabezpieczenia głównego logowania?

## Perspektywa frontendowa

React route guards, ukryte komponenty, redirecty, wyłączone przyciski i walidacja klienta wspierają UX, ale nie są granicą bezpieczeństwa uwierzytelniania.

Przeglądarka i wszystkie przesyłane przez nią wartości są pod kontrolą użytkownika. Backend musi weryfikować credentials, reset tokeny, ukończenie MFA, sesje i dostęp do chronionych endpointów.

## Ukończone wzorce praktyczne

### Reset token niepowiązany z kontem docelowym

Poprawny reset token wystawiony dla jednego konta został zaakceptowany, podczas gdy kontrolowany przez klienta `username` wskazał inne konto. Hasło drugiego konta zostało zmienione, a nowe hasło pozwoliło się na nie zalogować.

### MFA pokazane, ale niewymagane przez backend

Po poprawnej weryfikacji loginu i hasła aplikacja pokazała ekran MFA. Sesja mogła jednak uzyskać dostęp do `/my-account` przed wysłaniem kodu. Backend nie egzekwował stanu ukończenia MFA na chronionym zasobie.

## Powiązane istniejące notatki

Nie duplikuję wcześniejszych materiałów o enumeracji i authentication bypass:

- [Authentication Bypass and Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)

## Linki do nauki

- [OWASP A07:2025 - Authentication Failures](https://owasp.org/Top10/2025/A07_2025-Authentication_Failures/)
- [TryHackMe - OWASP Top 10 2025: IAAA Failures](https://tryhackme.com/room/owasptopten2025one)
- [PortSwigger - Authentication vulnerabilities](https://portswigger.net/web-security/authentication)
- [PortSwigger Lab - Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)
- [PortSwigger Lab - 2FA simple bypass](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)

## Rola plików

- `01-overview.md` jest głównym źródłem wiedzy dla omówionych tematów A07.
- `02-labs-or-practice.md` indeksuje ukończoną praktykę bez powtarzania całych dowodów.
- `03-checklist.md` służy do review developerskiego/AppSec.
- `04-regression-tests.md` zawiera testy weryfikujące poprawki.
- `05-learning-notes.md` zapisuje mój tok rozumowania, pomyłki i korekty.
- `labs-and-practice/` zawiera szczegółowe dowody z ćwiczeń.
- `security-findings/` zawiera przykładowe znaleziska dla developerów.
