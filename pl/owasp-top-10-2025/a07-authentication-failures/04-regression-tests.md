# Testy Regresyjne Authentication

Dobieraj testy do funkcji i findingu. Nie każda aplikacja potrzebuje każdego testu.

## Password Reset i Recovery

### Walidacja tokenu

- Brak tokenu jest odrzucany.
- Pusty token jest odrzucany.
- Zmieniony token jest odrzucany.
- Wygasły token jest odrzucany.
- Użyty token jest odrzucany.
- Replay po udanym resecie jest odrzucany.

### Powiązanie z kontem

- Token użytkownika A nie może zresetować użytkownika B.
- Zmiana client-controlled username nie zmienia celu resetu.
- Usunięcie username nie wywołuje niebezpiecznego fallbacku.
- Serwer ustala konto z zaufanego stanu tokenu.
- Token wystawiony do jednego celu nie działa dla innej akcji konta.

### Cykl życia

- Udany reset natychmiast unieważnia token.
- Istniejące sesje są odwołane zgodnie z wymaganiem.
- Stare hasło nie działa po resecie.
- Nowe hasło działa wyłącznie dla zamierzonego konta.
- Response resetu pozostaje generyczny dla istniejących i nieistniejących kont.

## MFA

### Egzekwowanie stanu

- Po haśle stan sesji to `mfa_pending`, nie fully authenticated.
- `/my-account` jest odrzucane lub przekierowane przed MFA.
- Chronione API jest odrzucane przed MFA.
- Direct navigation wokół ekranu MFA nie daje dostępu.
- Alternatywne chronione trasy sprawdzają ten sam stan.

### Obsługa kodu

- Błędny kod jest odrzucany.
- Wygasły kod jest odrzucany.
- Kod użytkownika A nie kończy próby użytkownika B.
- Nadmierna liczba prób uruchamia throttling.
- Poprawny kod zmienia stan na `mfa_completed`.
- Ponowne użycie kodu odpowiada oczekiwanej polityce jednorazowości.

### Recovery i zarządzanie

- Backup/recovery nie omija wymaganego dowodu.
- Wyłączenie MFA wymaga recent authentication.
- Trusted-device token wygasa i może zostać unieważniony.

## Session Management

### Rotacja

- Zapisz session ID przed loginem.
- Zaloguj się poprawnie.
- Potwierdź zmianę identyfikatora po authentication.
- Potwierdź rotację po privilege elevation lub step-up authentication.

### Logout i invalidation

- Zapisz authenticated session cookie.
- Wykonaj normalny logout.
- Użyj starego cookie na chronionym endpointcie.
- Potwierdź odrzucenie przez backend.
- Powtórz dla pozostałych typów sesji/tokenów.

### Zdarzenia hasłowe

- Zmień hasło i sprawdź stare sesje zgodnie z wymaganiami.
- Zresetuj hasło i sprawdź stare sesje.
- Wyłącz konto i potwierdź odwołanie aktywnych sesji.

### Timeout

- Potwierdź idle timeout.
- Potwierdź absolute timeout.
- Potwierdź czas remember-me.
- Potwierdź, że wygasłe tokeny fail closed.

## Enumeration i Credential Attack Controls

- Porównaj existing-user/wrong-password z non-existing-user.
- Porównaj message, status, response length i timing.
- Przetestuj rate limiting głównego login endpointu.
- Przetestuj alternatywne endpointy API/mobile.
- Sprawdź, czy zmiana source IP łatwo omija ochronę.
- Sprawdź, czy account lock można wykorzystać do denial of service.
- Potwierdź logowanie i alertowanie podejrzanych wzorców.

## Spójność Frontend/API

- Omiń nawigację React i wywołaj chronione API bezpośrednio.
- Zmień wskaźniki authentication w localStorage/sessionStorage.
- Potwierdź, że browser state nie tworzy uwierzytelnienia.
- Usuń lub zmień przesyłane pola tożsamości.
- Potwierdź poprawne decyzje backendu po pominięciu flow UI.
