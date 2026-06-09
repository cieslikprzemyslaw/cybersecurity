# Ukończenie MFA Nie Jest Egzekwowane na Chronionych Zasobach

## Summary

Po weryfikacji username i hasła aplikacja tworzy sesję, która może uzyskać dostęp do chronionych zasobów przed ukończeniem drugiego czynnika.

## Severity

**High** w przetestowanym scenariuszu, ponieważ atakujący z przejętymi credentials może ominąć kontrolę, która miała chronić konto przed dostępem przy użyciu samego hasła.

## STRIDE

**Spoofing** — aplikacja akceptuje atakującego jako fully authenticated bez wszystkich wymaganych dowodów tożsamości.

## Affected Flow

Two-factor authentication i dostęp do chronionego konta.

## Preconditions

Atakujący posiada poprawny username i hasło, ale nie posiada kodu MFA.

## Evidence

Bez wysłania kodu MFA sesja po weryfikacji hasła uzyskała dostęp do `/my-account` jako ofiara.

## Reproduction

1. Wyślij poprawny username i hasło ofiary.
2. Zatrzymaj się na ekranie MFA bez kodu.
3. Użyj bieżącej sesji do chronionego zasobu.
4. Zaobserwuj zwrócenie strony konta.

## Root Cause

Backend nie rozróżnia lub nie egzekwuje stanów `mfa_pending` i `mfa_completed` na chronionych zasobach.

## Impact

Skradzione lub ponownie użyte credentials wystarczają do wejścia na konto mimo widocznie włączonego MFA.

## Security Requirement

> Każda chroniona trasa i API musi wymagać fully authenticated server-side session z ukończonymi wszystkimi obowiązkowymi czynnikami.

## Remediation

- Utwórz jawny, ograniczony stan pre-auth/MFA-pending.
- Pozwól mu używać tylko endpointów kończących authentication.
- Egzekwuj MFA-completed na każdej chronionej trasie i API.
- Powiąż kod z użytkownikiem, sesją i próbą.
- Stosuj expiry i rate limiting.
- Bezpiecznie rotuj lub aktualizuj sesję po poprawnym MFA.

## Testy regresyjne

- Chronione strony odrzucają MFA-pending session.
- Chronione API odrzuca MFA-pending session.
- Direct navigation nie omija MFA.
- Błędne lub wygasłe kody nie tworzą pełnej sesji.
- Tylko udane MFA przechodzi do fully authenticated state.
