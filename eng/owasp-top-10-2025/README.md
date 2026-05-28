# OWASP Top 10 2025 Mapping Sprint

This directory maps my practical AppSec learning to the OWASP Top 10 2025 categories.

The goal is not to create theory-only notes. Each category should produce practical, GitHub-ready documentation that connects:

- what the OWASP category means,
- what I practised in labs or review tasks,
- what I would check during a developer/AppSec review,
- how I would report the issue,
- how I would prevent regression after a fix.

## Definition of Done for each OWASP category

A category is complete only when it has:

- `01-overview.md`
- `02-labs-or-practice.md`
- `03-checklist.md`
- `04-regression-tests.md`
- optional `05-learning-notes.md` for longer personal consolidation
- `security-findings/01-example-finding.md`
- an entry in `coverage-matrix.md`

## Current status

| Category | Status | Notes |
|---|---:|---|
| [A01 Broken Access Control](a01-broken-access-control/01-overview.md) | Completed after lab review | Includes a PortSwigger multi-step access control lab, checklist, example finding and regression tests. |
| [A02 Security Misconfiguration](a02-security-misconfiguration/01-overview.md) | Completed after two practice reviews | Includes a TryHackMe verbose API error disclosure task, a PortSwigger exposed `phpinfo()` debug page lab, learning notes, checklist, two findings and regression tests. |
| [A03 Software Supply Chain Failures](a03-software-supply-chain-failures/01-overview.md) | Completed after package and CI/CD review | Includes an npm package review, lifecycle script notes, lockfile/reproducibility review, learning notes, CI/CD checklist, example finding and regression checks. |
| [A04 Cryptographic Failures](a04-cryptographic-failures/01-overview.md) | Completed after two authentication-token labs | Includes PortSwigger weak remember-me cookie labs, Base64/MD5 token review, checklist, learning notes, example finding and regression tests. |
| A05: Injection | Planned | Existing SQLi notes can be mapped here. |
| A06: Insecure Design | Planned | Practical design-review exercise needed. |
| A07: Authentication Failures | Planned | Existing authentication bypass and username enumeration notes can be mapped here. |
| A08: Software or Data Integrity Failures | Planned | Practical review task: unsafe updates, CI/CD trust, package integrity. |
| A09: Security Logging and Alerting Failures | Planned | Practical logging and alerting review task. |
| A10: Mishandling of Exceptional Conditions | Planned | Practical error-handling and edge-case review task. |

## Related internal notes

- [Authentication Bypass / Username Enumeration](../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Broken Access Control / IDOR](../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [CSRF + SameSite Cookies](../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Path Traversal / File Access Bugs](../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [File Upload Vulnerabilities](../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [SQL Injection](../key-web-vulnerabilities/04-sql-injection/README.md)
- [XSS](../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
