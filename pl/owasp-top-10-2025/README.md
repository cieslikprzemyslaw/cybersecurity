# OWASP Top 10 2025 Mapping Sprint

Ten katalog mapuje moją praktyczną naukę AppSec na kategorie OWASP Top 10 2025.

Celem nie jest tworzenie notatek wyłącznie teoretycznych. Każda kategoria powinna prowadzić do praktycznej dokumentacji gotowej do GitHuba, która łączy:

- co oznacza dana kategoria OWASP,
- co ćwiczyłem w labach lub zadaniach review,
- co sprawdziłbym podczas review developerskiego/AppSec,
- jak opisałbym znalezisko,
- jak zapobiec regresji po wdrożeniu poprawki.

## Definition of Done dla każdej kategorii OWASP

Kategoria jest kompletna dopiero wtedy, gdy zawiera:

- `01-overview.md`
- `02-labs-or-practice.md`
- `03-checklist.md`
- `04-regression-tests.md`
- opcjonalny `05-learning-notes.md` dla dłuższej konsolidacji nauki
- `security-findings/01-example-finding.md`
- wpis w `coverage-matrix.md`

## Aktualny status

| Kategoria | Status | Notatki |
|---|---:|---|
| [A01 Broken Access Control](a01-broken-access-control/01-overview.md) | Ukończone po review laba | Zawiera wieloetapowy lab PortSwigger dotyczący access control, checklistę, przykładowe znalezisko i testy regresji. |
| [A02 Security Misconfiguration](a02-security-misconfiguration/01-overview.md) | Ukończone po dwóch review praktycznych | Zawiera zadanie TryHackMe dotyczące verbose API error disclosure, lab PortSwigger z wystawionym `phpinfo()`, learning notes, checklistę, dwa znaleziska i testy regresji. |
| [A03 Software Supply Chain Failures](a03-software-supply-chain-failures/01-overview.md) | Ukończone po review paczek i CI/CD | Zawiera review npm package, lifecycle scripts, lockfile/reproducibility, learning notes, checklistę CI/CD, przykładowe znalezisko i testy regresji. |
| [A04 Cryptographic Failures](a04-cryptographic-failures/01-overview.md) | Ukończone po dwóch labach tokenów uwierzytelniających | Zawiera laby PortSwigger o słabych remember-me cookies, review Base64/MD5, checklistę, learning notes, przykładowe znalezisko i testy regresji. |
| [A05: Injection](a05-injection/01-overview.md) | Ukończone po praktyce SQLi, NoSQL, OS Command Injection i SSTI | Zawiera SQL Injection, NoSQL Injection, OS Command Injection, SSTI, mapowanie XSS, Prompt Injection awareness, powiązane laby, checklistę, learning notes, findingi i testy regresji. |
| [A06: Insecure Design](a06-insecure-design/01-overview.md) | Ukończone po dwóch labach business logic i threat modelling | Zawiera cenę kontrolowaną przez klienta, nadużycie workflow kuponów, DFD w draw.io, 18 zagrożeń STRIDE, checklistę, przykładowy finding i testy regresji. |
| A07: Authentication Failures | Planowane | Istniejące notatki o authentication bypass i username enumeration można zmapować tutaj. |
| A08: Software or Data Integrity Failures | Planowane | Zadanie praktyczne: niebezpieczne aktualizacje, zaufanie w CI/CD, integralność pakietów. |
| A09: Security Logging and Alerting Failures | Planowane | Praktyczne zadanie review logowania i alertowania. |
| A10: Mishandling of Exceptional Conditions | Planowane | Praktyczne zadanie review obsługi błędów i edge case'ów. |

## Powiązane notatki wewnętrzne

- [Authentication Bypass / Username Enumeration](../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Broken Access Control / IDOR](../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [CSRF + SameSite Cookies](../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Path Traversal / File Access Bugs](../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [File Upload Vulnerabilities](../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [SQL Injection](../key-web-vulnerabilities/04-sql-injection/README.md)
- [XSS](../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [OWASP Top 10 for LLM Applications 2025](../owasp-top-10-for-llm-applications-2025/README.md)
