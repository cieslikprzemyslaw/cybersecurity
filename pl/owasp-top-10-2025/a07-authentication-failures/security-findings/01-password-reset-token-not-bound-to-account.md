# Password Reset Token Niepowiązany z Kontem Docelowym

## Summary

Endpoint resetu akceptuje poprawny reset token, jednocześnie pozwalając osobnemu client-controlled `username` wskazać inne konto. Serwer nie sprawdza, czy token został wydany dla konta, którego hasło zmienia.

## Severity

**High** w przetestowanym scenariuszu, ponieważ podatność doprowadziła do potwierdzonego account takeover.

W realnej aplikacji finalna ocena powinna uwzględniać uprawnienia konta, zachowanie MFA/recovery, session revocation, dostępność username'ów i compensating controls.

## STRIDE

**Spoofing** — atakujący może zmienić hasło ofiary i uwierzytelnić się jako ona.

## Affected Flow

Password reset / account recovery.

## Preconditions

- Atakujący może otrzymać poprawny reset token dla własnego konta.
- Atakujący zna lub potrafi ustalić username ofiary.

## Evidence

Token wystawiony dla jednego użytkownika wysłano z username innego użytkownika. Serwer zaakceptował reset, a nowe hasło pozwoliło zalogować się na konto ofiary.

## Reproduction

1. Poproś o reset dla konta kontrolowanego przez atakującego.
2. Odbierz poprawny reset token.
3. Wyślij request resetu, zmieniając username na ofiarę.
4. Ustaw nowe hasło.
5. Zaloguj się na konto ofiary nowym hasłem.

## Root Cause

Reset token nie jest bezpiecznie powiązany z kontem docelowym. Serwer ufa client-controlled identity field podczas wrażliwej zmiany stanu.

## Impact

Atakujący może przejąć inne konto. Uprzywilejowany cel może prowadzić do administrative compromise i dostępu do wrażliwych danych lub funkcji.

## Security Requirement

> Serwer musi ustalać cel resetu na podstawie zaufanego stanu tokenu i odrzucać próbę użycia tokenu dla innego konta.

## Remediation

- Przechowuj server-side mapping token -> user, purpose, expiry i use status.
- Zweryfikuj token przed zmianą hasła.
- Ustal użytkownika z tokenu, a nie z request username.
- Token powinien być nieprzewidywalny, krótkotrwały i jednorazowy.
- Unieważnij token po sukcesie.
- Przejrzyj revocation sesji po resecie.
- Używaj generycznych odpowiedzi recovery.

## Testy regresyjne

- Token A nie resetuje B.
- Zmieniony username nie retargetuje operacji.
- Pusty, brakujący, nieprawidłowy, wygasły i użyty token jest odrzucany.
- Replay tokenu jest odrzucany.
- Udany reset dotyczy tylko konta powiązanego z tokenem.
