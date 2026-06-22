# OWASP Top 10 2025 Mapping Sprint

This directory maps my practical AppSec learning to the OWASP Top 10 2025 categories.

The mapping sprint is completed for A01 through A10.

See the [learning summary](learning-summary.md) for the main takeaways, recurring patterns, and next steps from the sprint.

The goal is not to create theory-only notes. Each category should produce practical, GitHub-ready documentation that connects:

- what the OWASP category means,
- what I practised in labs or review tasks,
- what I would check during a developer/AppSec review,
- how I would report the issue,
- how I would prevent regression after a fix.

## Definition of Done for each OWASP category

A category is complete only when it has a usable topic structure, not just a placeholder row.

Minimum expected outputs:

- `README.md` as the topic index
- `01-overview.md`
- topic-specific theory or review notes when the subject needs them
- a checklist file such as `03-checklist.md` or `06-checklist.md`
- a regression-test file such as `04-regression-tests.md` or `07-regression-tests.md`
- optional learning notes for longer consolidation
- `labs-and-practice/` when practical review notes exist
- at least one example write-up under `security-findings/`
- an updated entry in `coverage-matrix.md`

## Current status

| Category | Status | Notes |
|---|---:|---|
| [A01 Broken Access Control](a01-broken-access-control/01-overview.md) | Completed after lab review | Includes a PortSwigger multi-step access control lab, checklist, example finding and regression tests. |
| [A02 Security Misconfiguration](a02-security-misconfiguration/01-overview.md) | Completed after two practice reviews | Includes a TryHackMe verbose API error disclosure task, a PortSwigger exposed `phpinfo()` debug page lab, learning notes, checklist, two findings and regression tests. |
| [A03 Software Supply Chain Failures](a03-software-supply-chain-failures/01-overview.md) | Completed after package and CI/CD review | Includes an npm package review, lifecycle script notes, lockfile/reproducibility review, learning notes, CI/CD checklist, example finding and regression checks. |
| [A04 Cryptographic Failures](a04-cryptographic-failures/01-overview.md) | Completed after two authentication-token labs | Includes PortSwigger weak remember-me cookie labs, Base64/MD5 token review, checklist, learning notes, example finding and regression tests. |
| [A05: Injection](a05-injection/01-overview.md) | Completed after SQLi, NoSQL, OS Command Injection, and SSTI practice | Includes SQL Injection, NoSQL Injection, OS Command Injection, SSTI, XSS mapping, Prompt Injection awareness, related labs, checklist, learning notes, findings, and regression tests. |
| [A06: Insecure Design](a06-insecure-design/01-overview.md) | Completed after two business-logic labs and threat modelling | Includes client-controlled pricing, coupon workflow abuse, a draw.io DFD, 18 STRIDE threats, checklist, example finding and regression tests. |
| [A07: Authentication Failures](a07-authentication-failures/01-overview.md) | Completed after recovery and MFA bypass labs | Includes TryHackMe IAAA/A07 review, PortSwigger password-reset broken logic and 2FA bypass labs, authentication checklist, learning notes, two findings and regression tests. |
| [A08: Software or Data Integrity Failures](a08-software-or-data-integrity-failures/README.md) | Completed after integrity and deserialization review | Includes integrity and authenticity notes, A03 versus A08 comparison, insecure deserialization review, checklist, regression tests, learning notes, labs-and-practice notes, and an example finding. |
| [A09: Security Logging and Alerting Failures](a09-security-logging-and-alerting-failures/README.md) | Completed after logging and alerting review | Includes A09 overview, detection and audit-log notes, monitoring and response guidance, sensitive-data review, frontend logging notes, checklist, regression tests, learning notes, labs-and-practice notes, and an example finding. |
| [A10: Mishandling of Exceptional Conditions](a10-mishandling-of-exceptional-conditions/README.md) | Completed after theory and practical review exercises | Includes OWASP A10 theory, OWASP Error Handling Cheat Sheet review, API error handling, partial-state and rollback review, malformed parameter review, timeout and retry review, duplicate submit and race-condition basics, checklist, learning notes, labs-and-practice notes, and an example finding. |

## Related internal notes

- [Authentication Bypass / Username Enumeration](../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Broken Access Control / IDOR](../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [CSRF + SameSite Cookies](../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Path Traversal / File Access Bugs](../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [File Upload Vulnerabilities](../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [SQL Injection](../key-web-vulnerabilities/04-sql-injection/README.md)
- [XSS](../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [OWASP Top 10 for LLM Applications 2025](../owasp-top-10-for-llm-applications-2025/README.md)
