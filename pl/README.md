# Indeks Notatek PL

Ten katalog jest polską wersją praktycznej ścieżki nauki Web AppSec.

Najpierw warto przejść przez fundamenty, potem threat modeling, a nastepnie moduły z konkretnymi podatnościami. Mapowanie OWASP Top 10 2025 przydaje się wtedy, gdy chcesz połączyć notatki z szerszym frameworkiem review AppSec.

## Zacznij tutaj

| Krok | Obszar | Użyj, gdy chcesz... |
|---:|---|---|
| 1 | [Fundamenty Web AppSec](fundamentals/README.md) | zbudować podstawy request/response, auth, Burpa i attack surface. |
| 2 | [Threat Modeling](threat-modeling/README.md) | identyfikowac zagrozenia projektowe, granice zaufania, pomysly na walidacje i mitygacje. |
| 3 | [Key Web Vulnerabilities](key-web-vulnerabilities/README.md) | uczyć się po jednej klasie podatności: notatki, checklisty i laby. |
| 4 | [OWASP Top 10 2025 Mapping Sprint](owasp-top-10-2025/README.md) | zmapować praktykę na kategorie OWASP i format review. |
| 5 | [OWASP Top 10 for LLM Applications 2025](owasp-top-10-for-llm-applications-2025/README.md) | przejrzeć Prompt Injection, RAG, agentów, tools i kontrole aplikacji LLM. |
| 6 | [Container Security](container-security/README.md) | przejrzeć Docker hardening, runtime images, Compose networking, wolumeny i troubleshooting. |

## Fundamenty Web AppSec

Podstawy przed przejściem do konkretnych podatności:

| Temat | Notatka |
|---|---|
| HTTP, request/response i podstawy auth | [Otwórz](fundamentals/01-http-request-response-auth.md) |
| Burp Suite, Proxy i podstawy Repeatera | [Otwórz](fundamentals/02-burp-suite-proxy-repeater.md) |
| Content discovery i attack surface | [Otwórz](fundamentals/03-content-discovery-attack-surface.md) |

## Threat Modeling

| Temat | Notatki |
|---|---|
| Indeks threat modelingu | [Otwórz](threat-modeling/README.md) |
| Simple web application threat model | [Otwórz](threat-modeling/simple-web-application-threat-model/README.md) |

## Key Web Vulnerabilities

| # | Temat | Główne notatki | Praktyka |
|---:|---|---|---|
| 01 | Authentication Bypass i Username Enumeration | [Indeks](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md), [Overview](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/overview.md), [Cheat Sheet](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/cheat-sheet.md), [Learning Summary](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/learning-summary.md) | [Laby](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/labs/) |
| 02 | Broken Access Control i IDOR | [Indeks](key-web-vulnerabilities/02-broken-access-control-idor/README.md), [Overview](key-web-vulnerabilities/02-broken-access-control-idor/overview.md), [Cheat Sheet](key-web-vulnerabilities/02-broken-access-control-idor/cheat-sheet.md), [Learning Summary](key-web-vulnerabilities/02-broken-access-control-idor/learning-summary.md) | [Laby](key-web-vulnerabilities/02-broken-access-control-idor/labs/) |
| 03 | Cross-Site Scripting | [Indeks](key-web-vulnerabilities/03-cross-site-scripting-xss/README.md), [Overview](key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md), [Cheat Sheet](key-web-vulnerabilities/03-cross-site-scripting-xss/cheat-sheet.md), [Wzorce frameworkowe](key-web-vulnerabilities/03-cross-site-scripting-xss/framework-risky-patterns/) | [Laby](key-web-vulnerabilities/03-cross-site-scripting-xss/labs/) |
| 04 | SQL Injection | [Indeks](key-web-vulnerabilities/04-sql-injection/README.md), [Overview](key-web-vulnerabilities/04-sql-injection/overview.md), [Cheat Sheet](key-web-vulnerabilities/04-sql-injection/cheat-sheet.md) | [Laby](key-web-vulnerabilities/04-sql-injection/labs/) |
| 05 | CSRF i SameSite Cookies | [Indeks](key-web-vulnerabilities/05-csrf-samesite-cookies/README.md), [Overview](key-web-vulnerabilities/05-csrf-samesite-cookies/overview.md), [Cheat Sheet](key-web-vulnerabilities/05-csrf-samesite-cookies/cheat-sheet.md) | [Laby](key-web-vulnerabilities/05-csrf-samesite-cookies/labs/) |
| 06 | Path Traversal i File Access Bugs | [Indeks](key-web-vulnerabilities/06-path-traversal-file-access/README.md), [Core Concepts](key-web-vulnerabilities/06-path-traversal-file-access/01-core-concepts.md), [Gdzie testować](key-web-vulnerabilities/06-path-traversal-file-access/02-where-to-test-cheatsheet.md), [Payloady](key-web-vulnerabilities/06-path-traversal-file-access/03-payloads-and-bypasses-cheatsheet.md), [LFI/RFI](key-web-vulnerabilities/06-path-traversal-file-access/04-lfi-rfi-cheatsheet.md), [Remediacja](key-web-vulnerabilities/06-path-traversal-file-access/05-remediation-cheatsheet.md) | [Laby](key-web-vulnerabilities/06-path-traversal-file-access/labs/) |
| 07 | File Upload Vulnerabilities | [Indeks](key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md), [Core Concepts](key-web-vulnerabilities/07-file-upload-vulnerabilities/01-core-concepts.md), [Anatomia requestu](key-web-vulnerabilities/07-file-upload-vulnerabilities/02-upload-request-anatomy.md), [Bypassy](key-web-vulnerabilities/07-file-upload-vulnerabilities/03-validation-and-filter-bypass-cheatsheet.md), [Metodyka testów](key-web-vulnerabilities/07-file-upload-vulnerabilities/04-black-box-testing-methodology.md), [Remediacja](key-web-vulnerabilities/07-file-upload-vulnerabilities/05-remediation-cheatsheet.md) | [Laby](key-web-vulnerabilities/07-file-upload-vulnerabilities/labs/), [Pliki testowe](key-web-vulnerabilities/07-file-upload-vulnerabilities/lab-test-files/README.md) |
| 08 | Server-Side Request Forgery | [Indeks](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md), [Podstawowe pojęcia](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/01-core-concepts.md), [Gdzie testować](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/02-where-to-test-cheatsheet.md), [Workflow](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/03-ssrf-testing-workflow.md), [Bypassy](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/04-bypasses-and-filtering-cheatsheet.md), [Remediacja](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/05-remediation-cheatsheet.md) | [Laby](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/labs/) |

## Container Security

| Temat | Notatki |
|---|---|
| Docker hardening | [Indeks](container-security/docker-hardening/00-index.md), [Mental Model](container-security/docker-hardening/01-container-mental-model.md), [Multi-stage Builds](container-security/docker-hardening/02-dockerfile-and-multi-stage-builds.md), [Runtime Images](container-security/docker-hardening/03-node-api-and-react-nginx-runtime.md), [Compose i Storage](container-security/docker-hardening/04-compose-volumes-networking-and-storage.md), [Troubleshooting](container-security/docker-hardening/05-runtime-troubleshooting.md), [Checklista](container-security/docker-hardening/06-hardening-checklist.md), [Lab](container-security/docker-hardening/07-appsec-report-builder-lab.md) |

## OWASP Top 10 2025 Mapping

| Zasób | Link |
|---|---|
| Indeks OWASP Top 10 2025 | [Otwórz](owasp-top-10-2025/README.md) |
| Coverage Matrix | [Otwórz](owasp-top-10-2025/coverage-matrix.md) |
| A01 Broken Access Control | [Overview](owasp-top-10-2025/a01-broken-access-control/01-overview.md), [Laby i praktyka](owasp-top-10-2025/a01-broken-access-control/02-labs-or-practice.md), [Checklista](owasp-top-10-2025/a01-broken-access-control/03-checklist.md), [Testy regresji](owasp-top-10-2025/a01-broken-access-control/04-regression-tests.md), [Przykładowe znalezisko](owasp-top-10-2025/a01-broken-access-control/security-findings/01-example-finding.md) |
| A02 Security Misconfiguration | [Overview](owasp-top-10-2025/a02-security-misconfiguration/01-overview.md), [Laby i praktyka](owasp-top-10-2025/a02-security-misconfiguration/02-labs-or-practice.md), [Checklista](owasp-top-10-2025/a02-security-misconfiguration/03-checklist.md), [Testy regresji](owasp-top-10-2025/a02-security-misconfiguration/04-regression-tests.md), [Znaleziska](owasp-top-10-2025/a02-security-misconfiguration/security-findings/) |
| A03 Software Supply Chain Failures | [Overview](owasp-top-10-2025/a03-software-supply-chain-failures/01-overview.md), [Laby i praktyka](owasp-top-10-2025/a03-software-supply-chain-failures/02-labs-or-practice.md), [Checklista](owasp-top-10-2025/a03-software-supply-chain-failures/03-checklist.md), [Testy regresji](owasp-top-10-2025/a03-software-supply-chain-failures/04-regression-tests.md), [Przykładowe znalezisko](owasp-top-10-2025/a03-software-supply-chain-failures/security-findings/01-example-finding.md) |
| A04 Cryptographic Failures | [Overview](owasp-top-10-2025/a04-cryptographic-failures/01-overview.md), [Laby i praktyka](owasp-top-10-2025/a04-cryptographic-failures/02-labs-or-practice.md), [Checklista](owasp-top-10-2025/a04-cryptographic-failures/03-checklist.md), [Testy regresji](owasp-top-10-2025/a04-cryptographic-failures/04-regression-tests.md), [Przykładowe znalezisko](owasp-top-10-2025/a04-cryptographic-failures/security-findings/01-example-finding.md) |
| A05 Injection | [Overview](owasp-top-10-2025/a05-injection/01-overview.md), [Laby i praktyka](owasp-top-10-2025/a05-injection/02-labs-or-practice.md), [Checklista](owasp-top-10-2025/a05-injection/03-checklist.md), [Testy regresji](owasp-top-10-2025/a05-injection/04-regression-tests.md), [Learning Notes](owasp-top-10-2025/a05-injection/05-learning-notes.md), [Znaleziska](owasp-top-10-2025/a05-injection/security-findings/) |
| A06 Insecure Design | [Overview](owasp-top-10-2025/a06-insecure-design/01-overview.md), [Laby i praktyka](owasp-top-10-2025/a06-insecure-design/02-labs-or-practice.md), [Checklista](owasp-top-10-2025/a06-insecure-design/03-checklist.md), [Testy regresji](owasp-top-10-2025/a06-insecure-design/04-regression-tests.md), [Learning Notes](owasp-top-10-2025/a06-insecure-design/05-learning-notes.md), [Przykładowe znalezisko](owasp-top-10-2025/a06-insecure-design/security-findings/01-client-controlled-product-price.md) |
| A07 Authentication Failures | [Overview](owasp-top-10-2025/a07-authentication-failures/01-overview.md), [Laby i praktyka](owasp-top-10-2025/a07-authentication-failures/02-labs-or-practice.md), [Checklista](owasp-top-10-2025/a07-authentication-failures/03-checklist.md), [Testy regresji](owasp-top-10-2025/a07-authentication-failures/04-regression-tests.md), [Learning Notes](owasp-top-10-2025/a07-authentication-failures/05-learning-notes.md), [Znaleziska](owasp-top-10-2025/a07-authentication-failures/security-findings/) |
| A08 Software or Data Integrity Failures | [Indeks](owasp-top-10-2025/a08-software-or-data-integrity-failures/README.md), [Overview](owasp-top-10-2025/a08-software-or-data-integrity-failures/01-overview.md), [Checklista](owasp-top-10-2025/a08-software-or-data-integrity-failures/06-checklist.md), [Testy regresji](owasp-top-10-2025/a08-software-or-data-integrity-failures/07-regression-tests.md), [Learning Notes](owasp-top-10-2025/a08-software-or-data-integrity-failures/08-learning-notes.md), [Laby i praktyka](owasp-top-10-2025/a08-software-or-data-integrity-failures/labs-and-practice/README.md), [Przykładowe znalezisko](owasp-top-10-2025/a08-software-or-data-integrity-failures/security-findings/01-example-finding-insecure-deserialization.md) |
| A09 Security Logging and Alerting Failures | [Indeks](owasp-top-10-2025/a09-security-logging-and-alerting-failures/README.md), [Overview](owasp-top-10-2025/a09-security-logging-and-alerting-failures/01-overview.md), [Checklista](owasp-top-10-2025/a09-security-logging-and-alerting-failures/06-checklist.md), [Testy regresji](owasp-top-10-2025/a09-security-logging-and-alerting-failures/07-regression-tests.md), [Learning Notes](owasp-top-10-2025/a09-security-logging-and-alerting-failures/08-learning-notes.md), [Laby i praktyka](owasp-top-10-2025/a09-security-logging-and-alerting-failures/labs-and-practice/README.md), [Przykładowe znalezisko](owasp-top-10-2025/a09-security-logging-and-alerting-failures/security-findings/01-example-finding-insufficient-security-logging.md) |
| A10 Mishandling of Exceptional Conditions | [Indeks](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/README.md), [Overview](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/01-overview.md), [Checklista](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/06-checklist.md), [Testy regresji](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/07-regression-tests.md), [Learning Notes](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/08-learning-notes.md), [Laby i praktyka](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/labs-and-practice/README.md), [Przykładowe znalezisko](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/security-findings/01-example-finding-insecure-exception-handling.md) |

## OWASP Top 10 for LLM Applications 2025

| Zasób | Link |
|---|---|
| Indeks bezpieczeństwa aplikacji LLM | [Otwórz](owasp-top-10-for-llm-applications-2025/README.md) |
| LLM01 Prompt Injection | [Otwórz](owasp-top-10-for-llm-applications-2025/llm01-prompt-injection/README.md) |

## Rekomendowana kolejność

1. Fundamenty 01-03.
2. Authentication bypass i username enumeration.
3. Broken access control i IDOR.
4. XSS.
5. SQL Injection.
6. CSRF i SameSite cookies.
7. Path Traversal i File Access Bugs.
8. File Upload Vulnerabilities.
9. Server-Side Request Forgery.
10. Mini-projekt threat modelingu.
11. Mapowanie OWASP Top 10 2025.
12. LLM01 Prompt Injection awareness.
13. Docker hardening i review runtime kontenerów.

## Rola plików

- `README.md` - indeks tematu, zakres i nawigacja.
- `overview.md` - zwięzłe źródło prawdy dla tematu.
- `cheat-sheet.md` - praktyczny workflow i checklista.
- `labs/*.md` - krótkie podsumowania legalnych labów.
- dłuższe summary - refleksja i głębsze utrwalenie tematu.
