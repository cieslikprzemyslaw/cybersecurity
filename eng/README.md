# English Notes Index

This directory is organised as a practical Web AppSec learning path.

Start with the fundamentals, then practise threat modeling, then work through the key vulnerability topics. Use the OWASP Top 10 2025 mapping when you want to connect the notes to a broader AppSec review framework.

## Start Here

| Step | Area | Use this when you want to... |
|---:|---|---|
| 1 | [Web AppSec Fundamentals](fundamentals/README.md) | build the request/response, auth, Burp and attack-surface basics. |
| 2 | [Threat Modeling](threat-modeling/README.md) | identify design-level threats, trust boundaries, validation ideas and mitigations. |
| 3 | [Key Web Vulnerabilities](key-web-vulnerabilities/README.md) | study one vulnerability class at a time with notes, checklists and labs. |
| 4 | [OWASP Top 10 2025 Mapping Sprint](owasp-top-10-2025/README.md) | map practical learning to OWASP categories and review outputs. |
| 5 | [OWASP Top 10 for LLM Applications 2025](owasp-top-10-for-llm-applications-2025/README.md) | review Prompt Injection, RAG, agents, tools and LLM application controls. |
| 6 | [Container Security](container-security/README.md) | review Docker hardening, runtime images, Compose networking, volumes and troubleshooting. |
| 7 | [CI/CD Security](ci-cd-security/README.md) | review GitHub Actions hardening, dependency security, SAST and repository protection. |

## Web AppSec Fundamentals

Foundation notes before vulnerability-specific study:

| Topic | Note |
|---|---|
| HTTP, requests, responses and auth basics | [Open](fundamentals/01-http-request-response-auth.md) |
| Burp Suite, Proxy and Repeater basics | [Open](fundamentals/02-burp-suite-proxy-repeater.md) |
| Content discovery and attack surface | [Open](fundamentals/03-content-discovery-attack-surface.md) |

## Threat Modeling

| Topic | Notes |
|---|---|
| Threat modeling index | [Open](threat-modeling/README.md) |
| Simple web application threat model | [Open](threat-modeling/simple-web-application-threat-model/README.md) |

## Key Web Vulnerabilities

| # | Topic | Core notes | Practice |
|---:|---|---|---|
| 01 | Authentication Bypass and Username Enumeration | [Index](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md), [Overview](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/overview.md), [Cheat Sheet](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/cheat-sheet.md), [Learning Summary](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/learning-summary.md) | [Labs](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/labs/) |
| 02 | Broken Access Control and IDOR | [Index](key-web-vulnerabilities/02-broken-access-control-idor/README.md), [Overview](key-web-vulnerabilities/02-broken-access-control-idor/overview.md), [Cheat Sheet](key-web-vulnerabilities/02-broken-access-control-idor/cheat-sheet.md), [Learning Summary](key-web-vulnerabilities/02-broken-access-control-idor/learning-summary.md) | [Labs](key-web-vulnerabilities/02-broken-access-control-idor/labs/) |
| 03 | Cross-Site Scripting | [Index](key-web-vulnerabilities/03-cross-site-scripting-xss/README.md), [Overview](key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md), [Cheat Sheet](key-web-vulnerabilities/03-cross-site-scripting-xss/cheat-sheet.md), [Framework Patterns](key-web-vulnerabilities/03-cross-site-scripting-xss/framework-risky-patterns/) | [Labs](key-web-vulnerabilities/03-cross-site-scripting-xss/labs/) |
| 04 | SQL Injection | [Index](key-web-vulnerabilities/04-sql-injection/README.md), [Overview](key-web-vulnerabilities/04-sql-injection/overview.md), [Cheat Sheet](key-web-vulnerabilities/04-sql-injection/cheat-sheet.md) | [Labs](key-web-vulnerabilities/04-sql-injection/labs/) |
| 05 | CSRF and SameSite Cookies | [Index](key-web-vulnerabilities/05-csrf-samesite-cookies/README.md), [Overview](key-web-vulnerabilities/05-csrf-samesite-cookies/overview.md), [Cheat Sheet](key-web-vulnerabilities/05-csrf-samesite-cookies/cheat-sheet.md) | [Labs](key-web-vulnerabilities/05-csrf-samesite-cookies/labs/) |
| 06 | Path Traversal and File Access Bugs | [Index](key-web-vulnerabilities/06-path-traversal-file-access/README.md), [Core Concepts](key-web-vulnerabilities/06-path-traversal-file-access/01-core-concepts.md), [Where to Test](key-web-vulnerabilities/06-path-traversal-file-access/02-where-to-test-cheatsheet.md), [Payloads](key-web-vulnerabilities/06-path-traversal-file-access/03-payloads-and-bypasses-cheatsheet.md), [LFI/RFI](key-web-vulnerabilities/06-path-traversal-file-access/04-lfi-rfi-cheatsheet.md), [Remediation](key-web-vulnerabilities/06-path-traversal-file-access/05-remediation-cheatsheet.md) | [Labs](key-web-vulnerabilities/06-path-traversal-file-access/labs/) |
| 07 | File Upload Vulnerabilities | [Index](key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md), [Core Concepts](key-web-vulnerabilities/07-file-upload-vulnerabilities/01-core-concepts.md), [Request Anatomy](key-web-vulnerabilities/07-file-upload-vulnerabilities/02-upload-request-anatomy.md), [Bypasses](key-web-vulnerabilities/07-file-upload-vulnerabilities/03-validation-and-filter-bypass-cheatsheet.md), [Testing](key-web-vulnerabilities/07-file-upload-vulnerabilities/04-black-box-testing-methodology.md), [Remediation](key-web-vulnerabilities/07-file-upload-vulnerabilities/05-remediation-cheatsheet.md) | [Labs](key-web-vulnerabilities/07-file-upload-vulnerabilities/labs/), [Test Files](key-web-vulnerabilities/07-file-upload-vulnerabilities/lab-test-files/README.md) |
| 08 | Server-Side Request Forgery | [Index](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md), [Core Concepts](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/01-core-concepts.md), [Where to Test](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/02-where-to-test-cheatsheet.md), [Workflow](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/03-ssrf-testing-workflow.md), [Bypasses](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/04-bypasses-and-filtering-cheatsheet.md), [Remediation](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/05-remediation-cheatsheet.md) | [Labs](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/labs/) |

## Container Security

| Topic | Notes |
|---|---|
| Docker hardening | [Index](container-security/docker-hardening/00-index.md), [Mental Model](container-security/docker-hardening/01-container-mental-model.md), [Multi-stage Builds](container-security/docker-hardening/02-dockerfile-and-multi-stage-builds.md), [Runtime Images](container-security/docker-hardening/03-node-api-and-react-nginx-runtime.md), [Compose and Storage](container-security/docker-hardening/04-compose-volumes-networking-and-storage.md), [Troubleshooting](container-security/docker-hardening/05-runtime-troubleshooting.md), [Checklist](container-security/docker-hardening/06-hardening-checklist.md), [Lab](container-security/docker-hardening/07-appsec-report-builder-lab.md) |

## CI/CD Security

| Topic | Notes |
|---|---|
| GitHub Actions hardening | [Index](ci-cd-security/github-actions-hardening/00-index.md), [Mental Model](ci-cd-security/github-actions-hardening/01-pipeline-mental-model.md), [Lab](ci-cd-security/github-actions-hardening/02-appsec-report-builder-lab.md), [Checklist](ci-cd-security/github-actions-hardening/03-checklist.md), [Evidence](ci-cd-security/github-actions-hardening/04-evidence.md) |

## OWASP Top 10 2025 Mapping

| Resource | Link |
|---|---|
| OWASP Top 10 2025 index | [Open](owasp-top-10-2025/README.md) |
| Coverage matrix | [Open](owasp-top-10-2025/coverage-matrix.md) |
| A01 Broken Access Control | [Overview](owasp-top-10-2025/a01-broken-access-control/01-overview.md), [Labs and Practice](owasp-top-10-2025/a01-broken-access-control/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a01-broken-access-control/03-checklist.md), [Regression Tests](owasp-top-10-2025/a01-broken-access-control/04-regression-tests.md), [Example Finding](owasp-top-10-2025/a01-broken-access-control/security-findings/01-example-finding.md) |
| A02 Security Misconfiguration | [Overview](owasp-top-10-2025/a02-security-misconfiguration/01-overview.md), [Labs and Practice](owasp-top-10-2025/a02-security-misconfiguration/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a02-security-misconfiguration/03-checklist.md), [Regression Tests](owasp-top-10-2025/a02-security-misconfiguration/04-regression-tests.md), [Findings](owasp-top-10-2025/a02-security-misconfiguration/security-findings/) |
| A03 Software Supply Chain Failures | [Overview](owasp-top-10-2025/a03-software-supply-chain-failures/01-overview.md), [Labs and Practice](owasp-top-10-2025/a03-software-supply-chain-failures/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a03-software-supply-chain-failures/03-checklist.md), [Regression Tests](owasp-top-10-2025/a03-software-supply-chain-failures/04-regression-tests.md), [Example Finding](owasp-top-10-2025/a03-software-supply-chain-failures/security-findings/01-example-finding.md) |
| A04 Cryptographic Failures | [Overview](owasp-top-10-2025/a04-cryptographic-failures/01-overview.md), [Labs and Practice](owasp-top-10-2025/a04-cryptographic-failures/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a04-cryptographic-failures/03-checklist.md), [Regression Tests](owasp-top-10-2025/a04-cryptographic-failures/04-regression-tests.md), [Example Finding](owasp-top-10-2025/a04-cryptographic-failures/security-findings/01-example-finding.md) |
| A05 Injection | [Overview](owasp-top-10-2025/a05-injection/01-overview.md), [Labs and Practice](owasp-top-10-2025/a05-injection/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a05-injection/03-checklist.md), [Regression Tests](owasp-top-10-2025/a05-injection/04-regression-tests.md), [Learning Notes](owasp-top-10-2025/a05-injection/05-learning-notes.md), [Findings](owasp-top-10-2025/a05-injection/security-findings/) |
| A06 Insecure Design | [Overview](owasp-top-10-2025/a06-insecure-design/01-overview.md), [Labs and Practice](owasp-top-10-2025/a06-insecure-design/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a06-insecure-design/03-checklist.md), [Regression Tests](owasp-top-10-2025/a06-insecure-design/04-regression-tests.md), [Learning Notes](owasp-top-10-2025/a06-insecure-design/05-learning-notes.md), [Example Finding](owasp-top-10-2025/a06-insecure-design/security-findings/01-client-controlled-product-price.md) |
| A07 Authentication Failures | [Overview](owasp-top-10-2025/a07-authentication-failures/01-overview.md), [Labs and Practice](owasp-top-10-2025/a07-authentication-failures/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a07-authentication-failures/03-checklist.md), [Regression Tests](owasp-top-10-2025/a07-authentication-failures/04-regression-tests.md), [Learning Notes](owasp-top-10-2025/a07-authentication-failures/05-learning-notes.md), [Findings](owasp-top-10-2025/a07-authentication-failures/security-findings/) |
| A08 Software or Data Integrity Failures | [Index](owasp-top-10-2025/a08-software-or-data-integrity-failures/README.md), [Overview](owasp-top-10-2025/a08-software-or-data-integrity-failures/01-overview.md), [Checklist](owasp-top-10-2025/a08-software-or-data-integrity-failures/06-checklist.md), [Regression Tests](owasp-top-10-2025/a08-software-or-data-integrity-failures/07-regression-tests.md), [Learning Notes](owasp-top-10-2025/a08-software-or-data-integrity-failures/08-learning-notes.md), [Labs and Practice](owasp-top-10-2025/a08-software-or-data-integrity-failures/labs-and-practice/README.md), [Example Finding](owasp-top-10-2025/a08-software-or-data-integrity-failures/security-findings/01-example-finding-insecure-deserialization.md) |
| A09 Security Logging and Alerting Failures | [Index](owasp-top-10-2025/a09-security-logging-and-alerting-failures/README.md), [Overview](owasp-top-10-2025/a09-security-logging-and-alerting-failures/01-overview.md), [Checklist](owasp-top-10-2025/a09-security-logging-and-alerting-failures/06-checklist.md), [Regression Tests](owasp-top-10-2025/a09-security-logging-and-alerting-failures/07-regression-tests.md), [Learning Notes](owasp-top-10-2025/a09-security-logging-and-alerting-failures/08-learning-notes.md), [Labs and Practice](owasp-top-10-2025/a09-security-logging-and-alerting-failures/labs-and-practice/README.md), [Example Finding](owasp-top-10-2025/a09-security-logging-and-alerting-failures/security-findings/01-example-finding-insufficient-security-logging.md) |
| A10 Mishandling of Exceptional Conditions | [Index](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/README.md), [Overview](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/01-overview.md), [Checklist](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/06-checklist.md), [Regression Tests](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/07-regression-tests.md), [Learning Notes](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/08-learning-notes.md), [Labs and Practice](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/labs-and-practice/README.md), [Example Finding](owasp-top-10-2025/a10-mishandling-of-exceptional-conditions/security-findings/01-example-finding-insecure-exception-handling.md) |

## OWASP Top 10 for LLM Applications 2025

| Resource | Link |
|---|---|
| LLM application security index | [Open](owasp-top-10-for-llm-applications-2025/README.md) |
| LLM01 Prompt Injection | [Open](owasp-top-10-for-llm-applications-2025/llm01-prompt-injection/README.md) |

## Recommended Reading Order

1. Fundamentals 01-03.
2. Authentication bypass and username enumeration.
3. Broken access control and IDOR.
4. XSS.
5. SQL injection.
6. CSRF and SameSite cookies.
7. Path traversal and file access bugs.
8. File upload vulnerabilities.
9. Server-side request forgery.
10. Threat modeling mini-project.
11. OWASP Top 10 2025 mapping.
12. LLM01 Prompt Injection awareness.
13. Docker hardening and container runtime review.
14. GitHub Actions and CI/CD security review.

## File Role Guide

- `README.md` - topic index, scope and navigation.
- `overview.md` - concise source of truth for the topic.
- `cheat-sheet.md` - practical workflow and checklist.
- `labs/*.md` - short summaries of legal training labs.
- longer summary files - personal reflection and deeper consolidation.
