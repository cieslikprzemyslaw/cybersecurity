# Ukończenie MFA Nie Jest Egzekwowane na Chronionych Zasobach

## Summary

Po weryfikacji nazwy użytkownika i hasła aplikacja tworzy sesję, która może uzyskać dostęp do chronionych zasobów przed ukończeniem drugiego czynnika.

## Severity

**High** w przetestowanym scenariuszu, ponieważ atakujący z przejętymi danymi logowania może ominąć kontrolę, która miała chronić konto przed dostępem przy użyciu samego hasła.

## STRIDE

**Spoofing** — aplikacja traktuje atakującego jako w pełni uwierzytelnionego bez wszystkich wymaganych dowodów tożsamości.

## Affected Flow

Uwierzytelnianie dwuskładnikowe i dostęp do chronionego konta.

## Preconditions

Atakujący zna poprawną nazwę użytkownika i hasło, ale nie posiada kodu MFA.

## Evidence

Bez wysłania kodu MFA sesja po weryfikacji hasła uzyskała dostęp do `/my-account` w imieniu ofiary.

## Reproduction

1. Wyślij poprawną nazwę użytkownika i hasło ofiary.
2. Zatrzymaj się na ekranie MFA bez podawania kodu.
3. Użyj bieżącej sesji do wysłania żądania do chronionego zasobu.
4. Zaobserwuj, że zostaje zwrócona strona konta.

## Root Cause

Backend nie rozróżnia lub nie egzekwuje stanów `mfa_pending` i `mfa_completed` dla chronionych zasobów.

## Impact

Skradzione lub ponownie użyte dane logowania wystarczają do wejścia na konto mimo widocznie włączonego MFA.

## Security Requirement

> Każda chroniona trasa i każde API musi wymagać w pełni uwierzytelnionej sesji po stronie serwera, z ukończonymi wszystkimi wymaganymi czynnikami.

## Remediation

- Utwórz jawny, ograniczony stan pre-auth / MFA-pending.
- Zezwól mu wyłącznie na korzystanie z endpointów kończących uwierzytelnianie.
- Egzekwuj stan MFA-completed na każdej chronionej trasie i w każdym API.
- Powiąż kod z użytkownikiem, sesją i konkretną próbą logowania.
- Stosuj wygasanie kodów i rate limiting.
- Bezpiecznie rotuj lub aktualizuj sesję po poprawnym MFA.

## Testy regresyjne

- Chronione strony odrzucają sesję w stanie MFA-pending.
- Chronione API odrzuca sesję w stanie MFA-pending.
- Bezpośrednia nawigacja nie omija MFA.
- Błędne lub wygasłe kody nie tworzą w pełni uwierzytelnionej sesji.
- Tylko udane MFA przechodzi do stanu fully authenticated.
