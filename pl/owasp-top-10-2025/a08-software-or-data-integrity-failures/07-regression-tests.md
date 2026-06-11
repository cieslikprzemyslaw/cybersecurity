# Testy regresyjne A08

Wybieraj testy istotne dla analizowanej funkcji.

## Serializowany stan klienta

### Zmieniona właściwość istotna z punktu widzenia bezpieczeństwa

Dla poprawnej sesji non-admin zmień `admin=false` na `admin=true`.

Oczekiwane:

- brak zaufanego stanu administratora wyłącznie na podstawie zmienionej wartości,
- `/admin` zwraca odmowę autoryzacji,
- akcje administracyjne są odrzucone,
- próba modyfikacji jest logowana, jeśli to właściwe.

### Zmieniona tożsamość lub konto

Zmień username, user ID, tenant ID lub account reference w stanie kontrolowanym przez klienta.

Oczekiwane:

- serwer ignoruje lub odrzuca zmienioną deklarację uprawnienia,
- sesja pozostaje powiązana z oryginalną zaufaną tożsamością.

### Brakujący lub błędny integrity value

Usuń lub uszkodź MAC/podpis.

Oczekiwane:

- bezpieczne odrzucenie,
- brak deserializacji i zaufanego przetwarzania,
- brak fallbacku do niepodpisanego stanu.

### Nieoczekiwany typ lub pole

Wyślij:

- nieoczekiwany typ obiektu,
- dodatkowe pola,
- malformed serialized data,
- pole z nieprawidłowym primitive type.

Oczekiwane:

- kontrolowane odrzucenie,
- brak niebezpiecznej konstrukcji obiektu,
- brak verbose stack trace,
- brak zmiany stanu.

### Replay

Użyj ponownie wygasłego albo wykorzystanego podpisanego obiektu.

Oczekiwane:

- odrzucenie przez expiry, nonce, session binding, action binding lub server-side revocation.

## Oprogramowanie i artefakty builda

### Zmiana artefaktu po buildzie

Zmień artefakt po zatwierdzeniu.

Oczekiwane:

- digest, signature lub provenance verification nie przechodzi,
- deployment zatrzymuje się.

### Brak podpisu

Dostarcz niepodpisaną aktualizację lub build tam, gdzie podpis jest wymagany.

Oczekiwane:

- instalacja lub deployment są blokowane.

### Niezaufane źródło

Dostarcz artefakt z niezatwierdzonego repozytorium, bucketu, registry albo URL.

Oczekiwane:

- odrzucenie przed wykonaniem lub deploymentem.

### Uszkodzony checksum

Zmień plik, pozostawiając oryginalny oczekiwany hash.

Oczekiwane:

- wykrycie mismatch,
- zatrzymanie operacji,
- logowanie błędu.

### Rollback albo replay

Spróbuj zainstalować starą, ale poprawnie podpisaną wersję, gdy rollback jest zabroniony.

Oczekiwane:

- odrzucenie przez version policy.

## Zewnętrzne zasoby przeglądarkowe

### Niezgodność SRI

Zmień skrypt na CDN bez aktualizacji `integrity`.

Oczekiwane:

- przeglądarka blokuje wykonanie,
- aplikacja bezpiecznie obsługuje brak zależności.

### Usunięte SRI

Usuń `integrity`, gdy policy go wymaga.

Oczekiwane:

- CI, linting, kontrola review lub polityka CSP wykrywa regresję.

### Niezatwierdzony origin

Zmień origin skryptu.

Oczekiwane:

- CSP lub polityka aplikacji blokuje ładowanie.

## Autoryzacja jako obrona w głąb

Nawet poprawnie podpisana wartość klienta nie powinna być jedyną granicą autoryzacji.

Test:

1. Wymuś wyświetlenie admin action w UI.
2. Wywołaj endpoint jako zwykły użytkownik.
3. Potwierdź odrzucenie przez serwer.

To udowadnia, że kontrola integralności klienta nie jest jedyną kontrolą dostępu.
