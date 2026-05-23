# Indeks Notatek PL

Ten katalog jest polską wersją ścieżki nauki z katalogu `eng`.

Najpierw warto przejść przez **fundamenty Web AppSec**, a potem moduły z serii **Key Web Vulnerabilities**.

## Fundamenty Web AppSec

Podstawy przed przejściem do konkretnych podatności:

- [HTTP, Request/Response i podstawy Auth](fundamentals/01-http-request-response-auth.md)
- [Burp Suite, Proxy i podstawy Repeatera](fundamentals/02-burp-suite-proxy-repeater.md)
- [Content Discovery i Attack Surface](fundamentals/03-content-discovery-attack-surface.md)

## [Key Web Vulnerabilities](key-web-vulnerabilities/)

Każdy temat zwykle zawiera:

- `README.md` - indeks i zakres tematu,
- `overview.md` - główną notatkę koncepcyjną,
- `cheat-sheet.md` - praktyczną checklistę, jeśli istnieje,
- `labs/` - krótkie podsumowania legalnych labów,
- `learning-summary.md` - dłuższe podsumowanie tematu, jeśli pasuje.

### 01. Authentication Bypass i Username Enumeration

- [Indeks tematu](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Overview](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/overview.md)
- [Learning Summary](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/learning-summary.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/cheat-sheet.md)
- [Laby](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/labs/)

### 02. Broken Access Control i IDOR

- [Indeks tematu](key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [Overview](key-web-vulnerabilities/02-broken-access-control-idor/overview.md)
- [Learning Summary](key-web-vulnerabilities/02-broken-access-control-idor/learning-summary.md)
- [Praktyczna checklista](key-web-vulnerabilities/02-broken-access-control-idor/cheat-sheet.md)
- [Laby](key-web-vulnerabilities/02-broken-access-control-idor/labs/)

### 03. Cross-Site Scripting

- [Indeks tematu](key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [Overview](key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/03-cross-site-scripting-xss/cheat-sheet.md)
- [Ryzykowne wzorce frameworkowe](key-web-vulnerabilities/03-cross-site-scripting-xss/framework-risky-patterns/)
- [Laby](key-web-vulnerabilities/03-cross-site-scripting-xss/labs/)

### 04. SQL Injection

- [Overview](key-web-vulnerabilities/04-sql-injection/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/04-sql-injection/cheat-sheet.md)
- [Laby](key-web-vulnerabilities/04-sql-injection/labs/)

### 05. CSRF i SameSite Cookies

- [Overview](key-web-vulnerabilities/05-csrf-samesite-cookies/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/05-csrf-samesite-cookies/cheat-sheet.md)
- [Laby](key-web-vulnerabilities/05-csrf-samesite-cookies/labs/)

### 06. Path Traversal i File Access Bugs

- [Indeks tematu](key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [Core Concepts](key-web-vulnerabilities/06-path-traversal-file-access/01-core-concepts.md)
- [Where to Test Checklist](key-web-vulnerabilities/06-path-traversal-file-access/02-where-to-test-cheatsheet.md)
- [Payloads and Bypasses Cheatsheet](key-web-vulnerabilities/06-path-traversal-file-access/03-payloads-and-bypasses-cheatsheet.md)
- [LFI / RFI Cheatsheet](key-web-vulnerabilities/06-path-traversal-file-access/04-lfi-rfi-cheatsheet.md)
- [Fixes / Review Cheatsheet](key-web-vulnerabilities/06-path-traversal-file-access/05-remediation-cheatsheet.md)
- [Laby](key-web-vulnerabilities/06-path-traversal-file-access/labs/)

### 07. File Upload Vulnerabilities

- [Indeks tematu](key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [Core Concepts](key-web-vulnerabilities/07-file-upload-vulnerabilities/01-core-concepts.md)
- [Upload Request Anatomy](key-web-vulnerabilities/07-file-upload-vulnerabilities/02-upload-request-anatomy.md)
- [Validation and Filter Bypass Cheatsheet](key-web-vulnerabilities/07-file-upload-vulnerabilities/03-validation-and-filter-bypass-cheatsheet.md)
- [Black-Box Testing Methodology](key-web-vulnerabilities/07-file-upload-vulnerabilities/04-black-box-testing-methodology.md)
- [Remediation Cheatsheet](key-web-vulnerabilities/07-file-upload-vulnerabilities/05-remediation-cheatsheet.md)
- [Laby](key-web-vulnerabilities/07-file-upload-vulnerabilities/labs/)
- [Pliki testowe do labów](key-web-vulnerabilities/07-file-upload-vulnerabilities/lab-test-files/README.md)

### 08. Server-Side Request Forgery

- [Indeks tematu](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [Podstawowe pojęcia](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/01-core-concepts.md)
- [Gdzie testować SSRF](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/02-where-to-test-cheatsheet.md)
- [Workflow testowania SSRF](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/03-ssrf-testing-workflow.md)
- [Bypassy i filtrowanie](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/04-bypasses-and-filtering-cheatsheet.md)
- [Remediacja](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/05-remediation-cheatsheet.md)
- [Laby](key-web-vulnerabilities/08-server-side-request-forgery-ssrf/labs/)

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
