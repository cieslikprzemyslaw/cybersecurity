# Przykładowe znalezisko: Brak autoryzacji na finalnym kroku potwierdzenia podniesienia roli

## Summary

Normalny uwierzytelniony użytkownik może podnieść własne konto do administratora przez bezpośrednie wysłanie finalnego requestu potwierdzającego w wieloetapowym workflow zarządzania rolami.

Pierwszy krok jest chroniony, ale finalny krok potwierdzenia zmieniający stan nie egzekwuje tego samego wymagania autoryzacji.

## Affected area

- Funkcja: Admin role management
- Endpoint: `POST /admin-roles`
- Dotknięta akcja: Podniesienie roli użytkownika
- Typ użytkownika: Normalny uwierzytelniony użytkownik
- Mapowanie OWASP: A01 Broken Access Control

## Severity

**High**

Użytkownik z niskimi uprawnieniami może podnieść swoje uprawnienia do administratora. Zależnie od zakresu funkcji admina może to ujawnić zarządzanie użytkownikami, wrażliwe dane lub inne uprzywilejowane operacje.

## Risk / Impact

Atakujący z normalnym kontem może ominąć chroniony pierwszy krok workflow podnoszenia roli i bezpośrednio wysłać finalny request potwierdzający.

Potencjalny impact:

- privilege escalation z normalnego użytkownika do administratora,
- dostęp do funkcji admin-only,
- nieautoryzowane zarządzanie użytkownikami,
- nieautoryzowane zmiany ról lub uprawnień,
- możliwy dostęp do wrażliwych danych lub akcji administracyjnych,
- utrata integralności modelu uprawnień.

## Root cause

Backend autoryzuje początkowy krok podniesienia roli, ale nie autoryzuje finalnego kroku potwierdzenia. Finalny endpoint wygląda, jakby traktował `confirmed=true` jako dowód, że użytkownik przeszedł zamierzony workflow admina.

To założenie jest niebezpieczne, ponieważ użytkownicy mogą wysyłać requesty backendowe bezpośrednio, bez używania UI.

## Evidence

### Request odrzucony na pierwszym kroku

Normalny uwierzytelniony użytkownik wysyła pierwszy request podniesienia roli:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

username=wiener&action=upgrade
```

Serwer odrzuca request:

```http
HTTP/2 401 Unauthorized
```

To pokazuje, że pierwszy krok ma access-control check.

### Finalny request potwierdzający zaakceptowany

Ten sam normalny uwierzytelniony użytkownik wysyła finalny request potwierdzający bezpośrednio:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
```

Serwer akceptuje request:

```http
HTTP/2 302 Found
Location: /admin
```

Użytkownik jest przekierowany do obszaru admina, co wskazuje na udane privilege escalation.

## Expected secure behaviour

Normalny uwierzytelniony użytkownik nie może wykonywać zmian ról przeznaczonych tylko dla administratorów.

Finalny request potwierdzający powinien zwrócić:

```http
HTTP/2 403 Forbidden
```

Aplikacja powinna też upewnić się, że:

- rola użytkownika pozostaje bez zmian,
- użytkownik nie może uzyskać dostępu do `/admin`,
- nieautoryzowana próba jest logowana, jeśli akcja jest uznana za wrażliwą,
- ta sama polityka autoryzacji dotyczy każdego kroku workflow.

## Remediation

Wdróż server-side authorization checks na każdym endpoincie zarządzania rolami, szczególnie na endpoincie wykonującym finalną akcję zmiany stanu.

Rekomendowane poprawki:

- Sprawdzaj rolę aktualnie uwierzytelnionego użytkownika przed przetworzeniem `POST /admin-roles`.
- Nie polegaj na `confirmed=true` jako dowodzie, że użytkownik przeszedł poprawny workflow admina.
- Nie ufaj `username`, `userId`, `role` ani podobnym parametrom kontrolowanym przez klienta przy decyzjach autoryzacyjnych.
- Stosuj deny-by-default dla akcji zarządzania rolami.
- Użyj współdzielonego authorization middleware, guarda, policy lub checku na poziomie service.
- Upewnij się, że finalny krok potwierdzenia wykonuje własny authorization check.
- Zwracaj `403 Forbidden`, gdy uwierzytelniony użytkownik nie ma wymaganych uprawnień.
- Dodaj testy regresji dla bezpośrednich requestów od normalnych użytkowników.

## Pomysły na testy regresji

Dodaj pokrycie regresyjne dla:

- normalnego użytkownika wysyłającego finalny request potwierdzający bezpośrednio,
- normalnego użytkownika próbującego podnieść rolę innego użytkownika,
- nieuwierzytelnionego użytkownika wysyłającego requesty zarządzania rolami,
- administratora wykonującego poprawny workflow zmiany roli,
- weryfikacji stanu po każdym odrzuconym requeście.

Szczegółowe przypadki testowe są opisane w [04-regression-tests.md](../04-regression-tests.md).

## Mapowanie OWASP

- OWASP Top 10 2025 A01 Broken Access Control
- Powiązany weakness pattern: vertical privilege escalation
- Powiązany obszar review: server-side authorization na endpointach zmieniających stan

## Developer takeaway

Każdy backendowy endpoint wykonujący wrażliwą akcję musi sam egzekwować autoryzację. Ochrona strony admina, pierwszego kroku workflow, frontend route albo widocznego przycisku nie wystarcza.

Użytkownicy mogą wysyłać requesty bezpośrednio, więc backend musi weryfikować uprawnienia w miejscu, w którym akcja jest wykonywana.
