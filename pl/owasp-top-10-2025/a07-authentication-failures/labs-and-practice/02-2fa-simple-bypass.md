# Lab: 2FA Simple Bypass

## Źródło

[PortSwigger Web Security Academy - 2FA simple bypass](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)

## Status

**PASS**

## Zamierzony flow authentication

1. Użytkownik przesyła username i hasło.
2. Backend weryfikuje pierwszy czynnik.
3. Backend tworzy ograniczony stan `mfa_pending`.
4. Użytkownik wysyła poprawny drugi czynnik.
5. Backend zmienia stan na `mfa_completed` lub fully authenticated.
6. Chronione strony i API stają się dostępne.

## Identity Claim

Username deklarował tożsamość Carlosa.

## Authentication Evidence

- Pierwszy czynnik: poprawne hasło.
- Drugi czynnik: kod MFA.

Zamierzony flow wymagał obu czynników.

## Obserwacja

Po poprawnym username i haśle aplikacja wyświetliła krok MFA.

Po przejściu na inną stronę wyglądało, jakby aplikacja rozpoznawała sesję jako Carlosa.

Było to podejrzane, ale sam UI nie był wystarczającym dowodem.

## Kontrolowany test

Bez wysłania kodu MFA użyto sesji utworzonej po haśle do żądania chronionej strony:

```http
GET /my-account
Cookie: session=<session-after-password-verification>
```

## Wynik i Evidence

Chroniona strona zwróciła konto Carlosa bez ukończenia drugiego czynnika.

Potwierdziło to brak egzekwowania stanu MFA po stronie backendu.

## Root Cause

Aplikacja utworzyła wystarczająco uprzywilejowaną sesję po weryfikacji hasła i nie sprawdzała ukończenia MFA na chronionych zasobach.

Problemem nie była sama możliwość zmiany URL. Direct navigation ujawniła brakującą kontrolę stanu po stronie serwera.

## Wymagany model stanu

```text
unauthenticated
    -> password_verified / mfa_pending
    -> fully_authenticated / mfa_completed
```

Stan `mfa_pending` nie może uzyskać dostępu do normalnych zasobów chronionych.

## STRIDE

**Spoofing** — atakujący posiadający hasło ofiary został zaakceptowany jako w pełni uwierzytelniony bez wymaganego drugiego czynnika.

## Impact

Atakujący, który zdobył poprawny username i hasło, może ominąć kontrolę chroniącą przed przejęciem credentials i wejść na konto ofiary.

## Security Requirement

> Serwer nie może traktować sesji jako fully authenticated przed weryfikacją wszystkich wymaganych czynników, a każda chroniona trasa i API musi sprawdzać stan MFA-completed.

## Remediation

Backend powinien:

- jawnie reprezentować ograniczony stan pre-MFA,
- pozwalać mu używać wyłącznie endpointów potrzebnych do dokończenia lub anulowania loginu,
- powiązać kod MFA z poprawnym użytkownikiem, sesją i próbą,
- egzekwować stan na wszystkich chronionych trasach i API,
- stosować expiry i rate limiting kodów,
- bezpiecznie rotować lub aktualizować sesję po MFA,
- chronić recovery, backup codes, trusted devices i flow wyłączania MFA.

## Testy regresyjne

- `/my-account` jest odrzucane przed MFA.
- Chronione API jest odrzucane przed MFA.
- Direct navigation wokół `/login2` nie daje dostępu.
- Błędny i wygasły kod nie kończy authentication.
- Kod jednego konta lub próby nie działa dla innego.
- Nadmierne próby MFA uruchamiają throttling.
- Udane MFA zmienia stan na fully authenticated.

## Finding Summary

> Atakujący posiadający prawidłowy username i hasło może ominąć MFA, ponieważ aplikacja udostępnia chronione zasoby przed zweryfikowaniem drugiego czynnika przez backend.
