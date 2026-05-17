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
- `learning-summary.md` - dłuższe podsumowanie nauki, jeśli pasuje.

### 01. Authentication Bypass i Username Enumeration

- [Indeks tematu](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Overview](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/overview.md)
- [Learning Summary](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/learning-summary.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/cheat-sheet.md)
- [Laby](key-web-vulnerabilities/01-authentication-bypass-username-enumeration/labs/)

### 02. Broken Access Control i IDOR

- [Overview](key-web-vulnerabilities/02-broken-access-control-idor/overview.md)

### 03. Cross-Site Scripting

- [Overview](key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md)

### 04. SQL Injection

- [Overview](key-web-vulnerabilities/04-sql-injection/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/04-sql-injection/cheat-sheet.md)
- [Laby](key-web-vulnerabilities/04-sql-injection/labs/)

### 05. CSRF i SameSite Cookies

- [Overview](key-web-vulnerabilities/05-csrf-samesite-cookies/overview.md)
- [Practical Cheat Sheet](key-web-vulnerabilities/05-csrf-samesite-cookies/cheat-sheet.md)
- [Laby](key-web-vulnerabilities/05-csrf-samesite-cookies/labs/)

## Rekomendowana kolejność

1. Fundamenty 01-03.
2. Authentication bypass i username enumeration.
3. Broken access control i IDOR.
4. XSS.
5. SQL Injection.
6. CSRF i SameSite cookies.
