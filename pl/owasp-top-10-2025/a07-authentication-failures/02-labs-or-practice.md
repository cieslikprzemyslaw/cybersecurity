# A07 Laby i praktyka

## Ukończona praca

| Źródło | Ćwiczenie | Główna lekcja | Status |
|---|---|---|---|
| TryHackMe | OWASP Top 10 2025 - wprowadzenie IAAA i sekcja A07 | Identification to deklaracja; authentication weryfikuje dowód; authorization sprawdza uprawnienia; accountability rejestruje aktywność | PASS |
| PortSwigger | Password reset broken logic | Dowód recovery musi być powiązany z kontem, a serwer nie może ufać kontrolowanemu `username` | PASS |
| PortSwigger | 2FA simple bypass | Stan po haśle musi pozostać ograniczony, dopóki backend nie potwierdzi MFA | PASS |
| Checkpoint mentorski | Sesje, logout, enumeracja, rate limiting | Wymagana jest rotacja i unieważnienie sesji po stronie serwera; generyczne błędy i warstwowy throttling ograniczają ataki na credentials | PASS po korekcie |

## Szczegółowe notatki

- [Password reset broken logic](labs-and-practice/01-password-reset-broken-logic.md)
- [2FA simple bypass](labs-and-practice/02-2fa-simple-bypass.md)
- [Podsumowanie praktyki](labs-and-practice/summary.md)

## Wcześniejsza powiązana praktyka

Username enumeration i authentication bypass zostały ukończone wcześniej, dlatego są linkowane zamiast duplikowane:

- [Authentication Bypass and Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)

## Uczciwy zakres

Poniższe tematy zostały omówione koncepcyjnie, ale nie były osobnymi labami A07 w tej sesji:

- session fixation,
- replay sesji po logout,
- concurrent sessions,
- remember-me design,
- bypass ochrony brute force,
- brute force kodów MFA,
- hard-coded i default credentials.

Są uwzględnione w checkliście i testach regresyjnych jako ważne obszary review, ale nie przedstawiam ich jako ukończonych dowodów z labów.
