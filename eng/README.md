# English Notes Index

This directory is organised as a learning path.

Start with **Web AppSec Fundamentals**, then move through the **Key Web Vulnerabilities** series topic by topic.

## Web AppSec Fundamentals

Foundation notes before vulnerability-specific study:

- [HTTP, Request/Response and Auth Basics](fundamentals/01-http-request-response-auth.md)
- [Burp Suite, Proxy and Repeater Basics](fundamentals/02-burp-suite-proxy-repeater.md)
- [Content Discovery and Attack Surface](fundamentals/03-content-discovery-attack-surface.md)

## [Key Web Vulnerabilities](key-web-vulnerabilities/)

Each vulnerability topic should usually contain:

- an `overview` note for the main concept,
- a `cheat sheet` for practical review,
- a `labs/` folder for short lab summaries,
- optional longer learning summaries.

### 01. Authentication Bypass and Username Enumeration

- [Topic Index](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Overview](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/overview.md)
- [Learning Summary](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/learning-summary.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/labs/)

### 02. Broken Access Control and IDOR

- [Topic Index](key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [Overview](key-web-vulnerabilities/02-broken-access-control-idor/overview.md)
- [Learning Summary](key-web-vulnerabilities/02-broken-access-control-idor/learning-summary.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/02-broken-access-control-idor/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/02-broken-access-control-idor/labs/)

### 03. Cross-Site Scripting

- [Topic Index](key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [Overview](key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/03-cross-site-scripting-xss/cheat-sheet.md)
- [Framework Risky Patterns](key-web-vulnerabilities/03-cross-site-scripting-xss/framework-risky-patterns/)
- [Labs](key-web-vulnerabilities/03-cross-site-scripting-xss/labs/)

### 04. SQL Injection

- [Overview](key-web-vulnerabilities/04-sql-injection/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/04-sql-injection/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/04-sql-injection/labs/)

### 05. CSRF and SameSite Cookies

- [Overview](key-web-vulnerabilities/05-csrf-samesite-cookies/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/05-csrf-samesite-cookies/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/05-csrf-samesite-cookies/labs/)

### 06. Path Traversal and File Access Bugs

- [Topic Index](key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [Core Concepts](key-web-vulnerabilities/06-path-traversal-file-access/01-core-concepts.md)
- [Where to Test Checklist](key-web-vulnerabilities/06-path-traversal-file-access/02-where-to-test-cheatsheet.md)
- [Payloads and Bypasses Cheatsheet](key-web-vulnerabilities/06-path-traversal-file-access/03-payloads-and-bypasses-cheatsheet.md)
- [LFI / RFI Cheatsheet](key-web-vulnerabilities/06-path-traversal-file-access/04-lfi-rfi-cheatsheet.md)
- [Fixes / Review Cheatsheet](key-web-vulnerabilities/06-path-traversal-file-access/05-remediation-cheatsheet.md)
- [Labs](key-web-vulnerabilities/06-path-traversal-file-access/labs/)

### 07. File Upload Vulnerabilities

- [Topic Index](key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [Core Concepts](key-web-vulnerabilities/07-file-upload-vulnerabilities/01-core-concepts.md)
- [Upload Request Anatomy](key-web-vulnerabilities/07-file-upload-vulnerabilities/02-upload-request-anatomy.md)
- [Validation and Filter Bypass Cheatsheet](key-web-vulnerabilities/07-file-upload-vulnerabilities/03-validation-and-filter-bypass-cheatsheet.md)
- [Black-Box Testing Methodology](key-web-vulnerabilities/07-file-upload-vulnerabilities/04-black-box-testing-methodology.md)
- [Remediation Cheatsheet](key-web-vulnerabilities/07-file-upload-vulnerabilities/05-remediation-cheatsheet.md)
- [Labs](key-web-vulnerabilities/07-file-upload-vulnerabilities/labs/)
- [Lab Test Files](key-web-vulnerabilities/07-file-upload-vulnerabilities/lab-test-files/README.md)

## Recommended Reading Order

1. Fundamentals 01-03.
2. Authentication bypass and username enumeration.
3. Broken access control and IDOR.
4. XSS.
5. SQL injection.
6. CSRF and SameSite cookies.
7. Path traversal and file access bugs.
8. File upload vulnerabilities.

## File Role Guide

- `overview.md` - concise source of truth for the topic.
- `cheat-sheet.md` - practical workflow and checklist.
- `labs/*.md` - short summaries of legal training labs.
- longer summary files - personal reflection and deeper consolidation.
