# A06: Insecure Design - Laby i praktyka

Ten plik indeksuje ukończoną legalną praktykę. Szczegółowe dowody znajdują się w podlinkowanych plikach i nie są tutaj powtarzane.

## Ukończona nauka

- TryHackMe: OWASP Top 10 2025 - Application Design Flaws
- PortSwigger: Excessive trust in client-side controls
- PortSwigger: Flawed enforcement of business rules
- Design i threat model fikcyjnej Event Booking Platform

## Szczegółowe notatki praktyczne

### Cena produktu kontrolowana przez klienta

[Excessive trust in client-side controls](labs-and-practice/01-excessive-trust-in-client-side-controls.md)

Główna lekcja:

> Serwer ufał cenie kontrolowanej przez klienta, mimo że powinien obliczać kwotę na podstawie zaufanych danych produktowych.

### Nadużycie workflow kuponów

[Flawed enforcement of business rules](labs-and-practice/02-flawed-enforcement-of-business-rules.md)

Główna lekcja:

> Dwa prawidłowe kupony i prawidłowe pojedyncze akcje stworzyły niedozwolony rezultat, ponieważ aplikacja nie egzekwowała reguły w pełnym stanie koszyka.

### Threat modelling

[Threat model fikcyjnej Event Booking Platform](labs-and-practice/03-fictional-event-platform-threat-model.md)

Ćwiczenie obejmowało:

- aktorów i zasoby,
- system context i application DFD,
- granice zaufania klient/serwer i system/wewnętrzny dostawca zewnętrzny,
- authentication, events, booking, payments i notifications,
- klasyfikację STRIDE z krótkim uzasadnieniem,
- mitygacje i testy weryfikujące,
- 18 priorytetowych zagrożeń.

## Aktualne ograniczenia

- Model jest fikcyjny i nie reprezentuje realnego systemu pracodawcy ani klienta.
- Severity zależy od kontekstu biznesowego, klasyfikacji danych i szczegółów wdrożenia.
- `Open` oznacza, że kontrola w fikcyjnym modelu nie została zaimplementowana i zweryfikowana; nie oznacza znalezienia realnej podatności.
- STRIDE wspiera review, ale nie zastępuje analizy business logic.
