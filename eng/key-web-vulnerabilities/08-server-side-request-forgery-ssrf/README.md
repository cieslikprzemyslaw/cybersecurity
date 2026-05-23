# Server-Side Request Forgery

This is the eighth topic in the Key Web Vulnerabilities series.

## Start Here

1. [Core Concepts](01-core-concepts.md)
2. [Where to Test Cheatsheet](02-where-to-test-cheatsheet.md)
3. [SSRF Testing Workflow](03-ssrf-testing-workflow.md)
4. [Bypasses and Filtering Cheatsheet](04-bypasses-and-filtering-cheatsheet.md)
5. [Remediation Cheatsheet](05-remediation-cheatsheet.md)
6. [PortSwigger Lab 01 - Basic SSRF Against the Local Server](labs/portswigger-01-basic-ssrf-localhost.md)
7. [PortSwigger Lab 02 - Basic SSRF Against Another Backend System](labs/portswigger-02-basic-ssrf-backend-system.md)
8. [PortSwigger Lab 03 - SSRF With Blacklist-Based Input Filter](labs/portswigger-03-blacklist-filter-bypass.md)

## Topic Focus

- user-controlled backend request destinations
- localhost and private-network access from server-side context
- regular vs blind SSRF
- URL, host, path and service-name parameters
- blacklist bypasses, encoding, redirects and parser confusion
- cloud metadata and link-local destination risk
- developer-focused remediation and regression testing

## Important Note

These notes are for legal labs, local practice and authorised testing only.

## File Roles

- `01-core-concepts.md` explains the core SSRF mental model.
- `02-where-to-test-cheatsheet.md` lists SSRF-prone features and parameters.
- `03-ssrf-testing-workflow.md` is the practical black-box testing workflow.
- `04-bypasses-and-filtering-cheatsheet.md` explains common weak-filter patterns.
- `05-remediation-cheatsheet.md` collects defensive fixes and regression ideas.
- `labs/` contains short legal lab summaries.
