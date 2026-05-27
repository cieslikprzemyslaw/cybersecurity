# English Notes Index

This directory is organised as a practical Web AppSec learning path.

Start with the fundamentals, then work through the key vulnerability topics. Use the OWASP Top 10 2025 mapping when you want to connect the notes to a broader AppSec review framework.

## Start Here

| Step | Area | Use this when you want to... |
|---:|---|---|
| 1 | [Web AppSec Fundamentals](fundamentals/README.md) | build the request/response, auth, Burp and attack-surface basics. |
| 2 | [Key Web Vulnerabilities](key-web-vulnerabilities/README.md) | study one vulnerability class at a time with notes, checklists and labs. |
| 3 | [OWASP Top 10 2025 Mapping Sprint](owasp-top-10-2025/README.md) | map practical learning to OWASP categories and review outputs. |

## Web AppSec Fundamentals

Foundation notes before vulnerability-specific study:

| Topic | Note |
|---|---|
| HTTP, requests, responses and auth basics | [Open](fundamentals/01-http-request-response-auth.md) |
| Burp Suite, Proxy and Repeater basics | [Open](fundamentals/02-burp-suite-proxy-repeater.md) |
| Content discovery and attack surface | [Open](fundamentals/03-content-discovery-attack-surface.md) |

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

## OWASP Top 10 2025 Mapping

| Resource | Link |
|---|---|
| OWASP Top 10 2025 index | [Open](owasp-top-10-2025/README.md) |
| Coverage matrix | [Open](owasp-top-10-2025/coverage-matrix.md) |
| A01 Broken Access Control | [Overview](owasp-top-10-2025/a01-broken-access-control/01-overview.md), [Labs and Practice](owasp-top-10-2025/a01-broken-access-control/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a01-broken-access-control/03-checklist.md), [Regression Tests](owasp-top-10-2025/a01-broken-access-control/04-regression-tests.md), [Example Finding](owasp-top-10-2025/a01-broken-access-control/security-findings/01-example-finding.md) |
| A02 Security Misconfiguration | [Overview](owasp-top-10-2025/a02-security-misconfiguration/01-overview.md), [Labs and Practice](owasp-top-10-2025/a02-security-misconfiguration/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a02-security-misconfiguration/03-checklist.md), [Regression Tests](owasp-top-10-2025/a02-security-misconfiguration/04-regression-tests.md), [Findings](owasp-top-10-2025/a02-security-misconfiguration/security-findings/) |
| A03 Software Supply Chain Failures | [Overview](owasp-top-10-2025/a03-software-supply-chain-failures/01-overview.md), [Labs and Practice](owasp-top-10-2025/a03-software-supply-chain-failures/02-labs-or-practice.md), [Checklist](owasp-top-10-2025/a03-software-supply-chain-failures/03-checklist.md), [Regression Tests](owasp-top-10-2025/a03-software-supply-chain-failures/04-regression-tests.md), [Example Finding](owasp-top-10-2025/a03-software-supply-chain-failures/security-findings/01-example-finding.md) |

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
10. OWASP Top 10 2025 mapping.

## File Role Guide

- `README.md` - topic index, scope and navigation.
- `overview.md` - concise source of truth for the topic.
- `cheat-sheet.md` - practical workflow and checklist.
- `labs/*.md` - short summaries of legal training labs.
- longer summary files - personal reflection and deeper consolidation.
