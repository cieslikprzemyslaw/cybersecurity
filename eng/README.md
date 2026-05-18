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

### 02. Authentication Bypass and Username Enumeration

- [Topic Index](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Overview](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/overview.md)
- [Learning Summary](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/learning-summary.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/labs/)

### 03. Broken Access Control and IDOR

- [Topic Index](key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [Overview](key-web-vulnerabilities/02-broken-access-control-idor/overview.md)
- [Learning Summary](key-web-vulnerabilities/02-broken-access-control-idor/learning-summary.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/02-broken-access-control-idor/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/02-broken-access-control-idor/labs/)

### 04. Cross-Site Scripting

- [Overview](key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md)

### 05. SQL Injection

- [Overview](key-web-vulnerabilities/04-sql-injection/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/04-sql-injection/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/04-sql-injection/labs/)

### 06. CSRF and SameSite Cookies

- [Overview](key-web-vulnerabilities/05-csrf-samesite-cookies/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/05-csrf-samesite-cookies/cheat-sheet.md)
- [Labs](key-web-vulnerabilities/05-csrf-samesite-cookies/labs/)

## Recommended Reading Order

1. Fundamentals 01-03.
2. Authentication bypass and username enumeration.
3. Broken access control and IDOR.
4. XSS.
5. SQL injection.
6. CSRF and SameSite cookies.

## File Role Guide

- `overview.md` - concise source of truth for the topic.
- `cheat-sheet.md` - practical workflow and checklist.
- `labs/*.md` - short summaries of legal training labs.
- longer summary files - personal reflection and deeper consolidation.
