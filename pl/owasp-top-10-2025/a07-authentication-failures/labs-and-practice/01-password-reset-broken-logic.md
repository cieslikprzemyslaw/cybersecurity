# Lab: Password Reset Broken Logic

## Źródło

[PortSwigger Web Security Academy - Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)

## Status

**PASS**

## Zamierzony flow authentication

1. Użytkownik przesyła identyfikator konta, aby rozpocząć recovery.
2. Backend tworzy reset token dla tego konta.
3. Token trafia przez kanał recovery użytkownika.
4. Użytkownik wysyła token i nowe hasło.
5. Backend weryfikuje token i resetuje tylko konto z nim powiązane.

## Identity Claim

Parametr `username` wybierał konto, którego hasło miało zostać zmienione.

## Oczekiwany dowód

`temp-forgot-password-token` był recovery evidence. Powinien potwierdzać prawo do resetu jednego konkretnego konta.

## Początkowa obserwacja

Request zawierał osobne wartości kontrolowane przez klienta:

```http
POST /forgot-password
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=<token-for-wiener>
&username=wiener
&new-password-1=<new-password>
&new-password-2=<new-password>
```

## Początkowa pomyłka

Najpierw zastanawiałem się, czy browser `session` cookie może uwierzytelnić mnie jako Carlosa.

Kontrolowany test pokazał, że cookie było tylko bieżącą sesją przeglądarki. Nie było reset tokenem i nie stawało się automatycznie authenticated session Carlosa.

## Hipoteza

Backend może sprawdzać, czy reset token istnieje, ale ufać osobnemu `username` przy wyborze konta docelowego.

## Kontrolowany test

Poprawny token pozostał bez zmian, a zmodyfikowano konto:

```http
POST /forgot-password
Content-Type: application/x-www-form-urlencoded

temp-forgot-password-token=<valid-token-for-wiener>
&username=carlos
&new-password-1=<new-password>
&new-password-2=<new-password>
```

## Wynik

Serwer zaakceptował request i wykonał redirect.

Był to dowód przetworzenia requestu, ale redirect sam nie potwierdzał jeszcze account takeover.

## Potwierdzenie impactu

Logowanie jako Carlos przy użyciu nowego hasła zakończyło się sukcesem.

Potwierdziło to:

- przyjęcie tokenu wystawionego dla Wienera,
- zmianę konta przez client-controlled username,
- zmianę hasła Carlosa,
- możliwość uwierzytelnienia jako Carlos.

## Root Cause

Backend nie powiązał bezpiecznie reset tokenu z kontem, którego hasło zmieniał. Zaufał przesłanemu przez klienta username.

Problemem nie było samo edytowalne hidden field. Wszystkie dane z przeglądarki są edytowalne.

## STRIDE

**Spoofing** — podatność pozwoliła zmienić hasło innego użytkownika i uwierzytelnić się jako on.

## Impact

Potwierdzony account takeover. Dla konta uprzywilejowanego mogłoby to oznaczać przejęcie funkcji administracyjnych i wrażliwych danych.

## Security Requirement

> Serwer musi ustalać konto docelowe na podstawie zaufanego stanu reset tokenu i nie może ufać client-controlled username przy wyborze konta, którego hasło jest zmieniane.

## Remediation

Przechowywać i weryfikować relację po stronie serwera:

```text
reset_token_hash
    -> user_id
    -> purpose=password_reset
    -> expires_at
    -> used_at
```

Serwer powinien:

- generować nieprzewidywalny token,
- przechowywać odpowiednio chronioną reprezentację, np. hash,
- ustalać użytkownika z tokenu,
- odrzucać tokeny nieprawidłowe, wygasłe i użyte,
- natychmiast unieważniać token po sukcesie,
- rozważyć revocation istniejących sesji,
- nie ujawniać istnienia kont w odpowiedziach recovery.

## Testy regresyjne

- Token A nie resetuje konta B.
- Zmieniony lub brakujący username nie retargetuje resetu.
- Brakujący, pusty, zmieniony, wygasły i użyty token jest odrzucany.
- Replay po udanym resecie jest odrzucany.
- Nowe hasło działa tylko dla konta powiązanego z tokenem.
- Istniejące sesje zachowują się zgodnie z wymaganiem revocation.

## Finding Summary

> Atakujący znający prawidłowy username może użyć własnego password-reset tokenu do resetu innego konta, ponieważ backend nie weryfikuje powiązania tokenu z wybranym użytkownikiem. Może to prowadzić do account takeover, w tym kont uprzywilejowanych.
