# Threat Model Fikcyjnej Event Booking Platform

## Cel

Ćwiczenie używało fikcyjnego systemu, aby model mógł zostać publicznie udostępniony bez ujawniania architektury pracodawcy, klienta ani informacji proprietary.

Celem było przećwiczenie lekkiego design review, a nie stworzenie pełnej architektury enterprise.

## Wybór narzędzia

Jeden duży diagram OWASP Threat Dragon stał się trudny do czytania.

Draw.io zostało użyte do:

- system context,
- application DFD,
- granice zaufania,
- znaczników threats na osobnej warstwie,
- custom threat metadata,
- strony threat register.

Threat Dragon nadal może być użyteczny dla mniejszych diagramów konkretnych workflow, ale pełna platforma była czytelniejsza w draw.io.

## Kontekst systemu

Aktorzy:

- Customer
- Event Organizer
- Platform Administrator
- External Payment Provider
- External Email Provider

Główne komponenty wewnętrzne:

- Web Frontend
- API Gateway
- Auth Service
- Event Service
- Booking Service
- Payment Service
- Notification Service
- User, Event, Booking i Payment databases

## Główne granice zaufania

### Client / Server Trust Boundary

Wszystko pochodzące z browsera jest traktowane jako możliwe do zmiany:

- IDs,
- roles,
- ceny,
- quantities,
- state fields,
- kolejność żądań,
- ponownie użyte żądania.

### Internal / Third-Party Trust Boundary

Payment i email providers są zależnościami zewnętrznymi.

Callbacki i delivery events muszą zostać uwierzytelnione, zwalidowane i zabezpieczone przed replay, zanim wpłyną na stan wewnętrzny.

## Kluczowe zasoby

- User identities, sessions, roles i account recovery
- Event ownership, visibility, capacity i publication state
- Bookings, ticket validity i attendee data
- Payment i refund state
- Poufność notification recipient i content
- Dostępność workflow booking, payment i notification

## Lekki workflow

1. Zidentyfikować zasoby.
2. Zidentyfikować actors.
3. Zmapować punkty wejścia i przepływy danych.
4. Oznaczyć granice zaufania.
5. Zapisać security assumptions.
6. Stworzyć realistyczne abuse cases.
7. Zidentyfikować istniejące i brakujące controls.
8. Zapisać testowalne security requirements.
9. Dodać testy weryfikujące.
10. Śledzić status i ownera.

## Użycie STRIDE

STRIDE zostało użyte jako klasyfikacja pomocnicza, nie jako całe ćwiczenie.

Każde zagrożenie zawiera:

- jedną główną kategorię STRIDE,
- krótkie uzasadnienie, dlaczego pasuje,
- affected element,
- description,
- mitigation,
- severity,
- status,
- weryfikacja.

Przykłady:

- `Spoofing`: skradziona sesja albo podrobiony webhook providera.
- `Tampering`: zmanipulowany stan eventu, bookingu albo payment amount.
- `Information Disclosure`: prywatny event albo nadmierne dane API.
- `Elevation of Privilege`: dostęp do cudzej rezerwacji lub wydarzenia.
- `Denial of Service`: notification flooding.

## Zakres zagrożeń

Finalny model zawiera 18 priorytetowych zagrożeń:

- 5 dotyczących authentication i sessions,
- 3 dotyczące event management,
- 3 dotyczące booking i tickets,
- 3 dotyczące payment i webhooks,
- 2 dotyczące notifications,
- 2 przekrojowe dotyczące API i authorization.

Pełne szczegóły znajdują się w [threat-register.md](threat-register.md).

## Znaczenie statusu

Wszystkie zagrożenia mają status `Open`.

Oznacza to, że kontrole w fikcyjnym modelu nie zostały zaimplementowane i zweryfikowane. Nie jest to twierdzenie, że realna aplikacja produkcyjna zawiera te podatności.

## Plik wizualny

[fictional-event-platform-complete-threat-model.drawio](fictional-event-platform-complete-threat-model.drawio)

Plik zawiera:

- `01-System-Context`
- `02-Application-DFD`
- `03-Threat-Register`
- warstwę `Architecture`
- warstwę `Threats`
- metadata dla `T-001` do `T-018`

## Główny rezultat nauki

Najbardziej użyteczny threat entry nie był najdłuższy.

Użyteczny wpis łączył:

```text
asset
  -> unsafe assumption
  -> abuse case
  -> missing control
  -> testable security requirement
  -> weryfikacja
```
