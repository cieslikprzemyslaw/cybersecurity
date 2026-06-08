# A06: Insecure Design

Ten katalog zawiera moje notatki do OWASP Top 10 2025 - A06: Insecure Design.

## Zacznij tutaj

- [Overview](01-overview.md)
- [Laby i praktyka](02-labs-or-practice.md)
- [Checklista design review](03-checklist.md)
- [Pomysły na testy regresji](04-regression-tests.md)
- [Learning notes](05-learning-notes.md)

## Zakres tematu

Ta sekcja obejmuje:

- security requirements i unsafe assumptions,
- design flaws versus błędy implementacji i misconfiguration,
- business logic i abuse cases,
- granice zaufania klient/serwer,
- egzekwowanie kontroli po stronie serwera,
- projektowanie workflow i przejść stanu,
- lekki threat modelling,
- STRIDE jako checklistę pomocniczą,
- testowalne mitygacje i sposoby weryfikacji.

## Ukończona praktyka

- [Excessive trust in client-side controls](labs-and-practice/01-excessive-trust-in-client-side-controls.md)
- [Flawed enforcement of business rules](labs-and-practice/02-flawed-enforcement-of-business-rules.md)
- [Threat model fikcyjnej Event Booking Platform](labs-and-practice/03-fictional-event-platform-threat-model.md)
- [Threat register](labs-and-practice/threat-register.md)
- [Podsumowanie praktyki](labs-and-practice/summary.md)

Model wizualny:

- [fictional-event-platform-complete-threat-model.drawio](labs-and-practice/fictional-event-platform-complete-threat-model.drawio)

## Bezpośrednie linki do nauki

- [OWASP A06:2025 - Insecure Design](https://owasp.org/Top10/2025/A06_2025-Insecure_Design/)
- [TryHackMe - Application Design Flaws](https://tryhackme.com/room/owasptopten2025two)
- [PortSwigger - Business logic vulnerabilities](https://portswigger.net/web-security/logic-flaws)
- [Lab: Excessive trust in client-side controls](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)
- [Lab: Flawed enforcement of business rules](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-flawed-enforcement-of-business-rules)

## Rola plików

- `01-overview.md` jest zwięzłym źródłem prawdy dla pojęć i rozróżnień A06.
- `02-labs-or-practice.md` indeksuje ukończoną legalną praktykę bez powtarzania pełnych dowodów.
- `03-checklist.md` jest praktyczną checklistą do design review.
- `04-regression-tests.md` zawiera wielokrotnego użytku pomysły na weryfikację poprawek.
- `05-learning-notes.md` zapisuje korekty i osobistą konsolidację nauki.
- `labs-and-practice/` zawiera szczegółowe dowody z labów i fikcyjny threat model.
- `security-findings/` zawiera przykładowy finding dla zespołu developerskiego.

## Przykładowy finding

- [Cena produktu kontrolowana przez klienta zaakceptowana przez checkout](security-findings/01-client-controlled-product-price.md)
