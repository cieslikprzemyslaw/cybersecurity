# A06: Insecure Design - Overview

## Czym jest Insecure Design?

Insecure Design występuje, gdy system lub workflow został zaprojektowany bez skutecznej kontroli wymaganej do ochrony assetu albo reguły biznesowej.

Problemem może być:

- brakująca kontrola bezpieczeństwa,
- kontrola umieszczona na złej granicy zaufania,
- niebezpieczne założenie o zachowaniu użytkownika,
- niepełny model workflow lub stanu,
- kontrola, która nie jest w stanie ochronić zamierzonej operacji.

Poprawnie napisany kod może dokładnie implementować niebezpieczny design.

## Główny mental model

```text
business requirement
  -> design decision
  -> trust assumption
  -> security control
  -> expected and unexpected behaviour
```

Pytania przydatne podczas review:

- Co system próbuje chronić?
- Kim są aktorzy?
- Co każdy aktor może zrobić?
- Które inputy i stany są kontrolowane przez klienta?
- Gdzie zmienia się poziom zaufania?
- Co musi być zawsze prawdą?
- Co nigdy nie może się wydarzyć?
- Co stanie się po pominięciu, powtórzeniu, zmianie kolejności lub równoległym wykonaniu kroku?
- Jaka kontrola musi zostać zaprojektowana przed implementacją?
- Jak sprawdzimy, że kontrola działa?

## Ważne rozróżnienia

| Problem | Znaczenie | Przykład |
|---|---|---|
| Insecure Design | Design nie zawiera skutecznej kontroli albo pozwala na niebezpieczny workflow | Klient przesyła autorytatywną cenę produktu |
| Implementation defect | Poprawna kontrola została zaprojektowana, ale kod wdrożył ją błędnie | Ownership check używa niewłaściwego porównania albo resource ID |
| Security Misconfiguration | Właściwy mechanizm istnieje, ale ma niebezpieczne ustawienia | Uwierzytelnianie jest wyłączone na produkcji przez konfigurację |
| Broken Access Control | Użytkownik uzyskuje dane lub akcje poza swoim zakresem | Organizer zmienia `eventId` i edytuje cudze wydarzenie |

Broken Access Control może wynikać z design flaw albo implementation defect. Samo obserwowane zachowanie nie zawsze dowodzi, na którym etapie powstał problem.

Brak testu lub brak dowodu nie jest automatycznie potwierdzeniem Insecure Design. Oznacza, że kontrola nie została jeszcze zweryfikowana.

## Frontend i granica zaufania

Kontrole frontendowe są przydatne dla:

- UX,
- szybkiego feedbacku,
- ograniczania przypadkowych błędów,
- ukrywania niedostępnych akcji,
- defence in depth.

Nie są główną granicą enforcementu.

Przykład:

```tsx
{isAdmin && <AdminPanel />}
```

Gdy `isAdmin` jest `false`, React nie renderuje komponentu do DOM. Poprawia to interfejs, ale nie dowodzi, że użytkownik nie może bezpośrednio wywołać API używanego przez komponent.

Backend nadal musi:

- ustalać tożsamość z poprawnej sesji lub tokenu,
- egzekwować role i ownership zasobu,
- sprawdzać stan workflow,
- obliczać autorytatywne wartości z zaufanych danych,
- zwracać wyłącznie dozwolone dane.

React route guard, disabled input albo hidden button kontrolują prezentację. Server-side authorization kontroluje realną możliwość wykonania akcji.

## Fakty, założenia, ryzyka i wymagania

### Fakt

Informacja potwierdzona przez architekturę, kod, request/response, udokumentowaną politykę albo obserwowane zachowanie.

### Założenie

Coś, co uważamy za prawdę, ale nie zostało jeszcze potwierdzone.

### Ryzyko

Co może się wydarzyć, jeżeli założenie jest błędne albo kontrola zawiedzie.

### Wymaganie bezpieczeństwa

Konkretna i testowalna reguła, którą system musi egzekwować.

Przykład:

```text
Fakt:
Request checkout zawiera cenę kontrolowaną przez klienta.

Założenie:
Backend ponownie oblicza cenę.

Ryzyko:
Klient może kupić produkt za zmanipulowaną kwotę.

Wymaganie bezpieczeństwa:
Serwer musi obliczać cenę produktu i sumę zamówienia z zaufanych danych produktowych po stronie serwera.
```

## Business logic i stan

Wiele problemów A06 używa pojedynczo prawidłowych wartości i akcji.

Niebezpieczny rezultat pojawia się przez:

- powtarzane akcje,
- zmianę kolejności kroków,
- pomijanie kroków,
- niedozwolone przejścia stanu,
- nieaktualny stan,
- równoległe żądania,
- zaufanie wartościom obliczonym przez klienta,
- niespójne reguły między trasami.

Input validation nie naprawi workflow, którego dozwolone przejścia stanu są błędne.

## Zasady secure design użyte w tym module

- Server-side enforcement
- Deny by default
- Least privilege
- Secure defaults
- Jawne maszyny stanów
- Obliczenia z zaufanych danych serwerowych
- Object-level authorization
- Idempotency
- Replay protection
- Fail-safe behaviour
- Minimal data disclosure
- Defence in depth
- Human approval albo recent authentication dla akcji wysokiego ryzyka

## Ukończone przykłady praktyczne

- Checkout ufał cenie produktu kontrolowanej przez klienta.
- Naprzemienne stosowanie prawidłowych kuponów ominęło zamierzony limit użycia.
- Fikcyjna Event Booking Platform została zamodelowana z aktorami, zasobami, przepływami danych, granicami zaufania, założeniami bezpieczeństwa i 18 priorytetowymi zagrożeniami.

Szczegółowe dowody znajdują się w `labs-and-practice/`, aby overview pozostał zwięzły.
