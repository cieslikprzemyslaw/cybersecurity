# A02:2025 - Security Misconfiguration: Labs and Practice

## Purpose

This file documents the practical work completed for A02:2025 - Security Misconfiguration.

The goal was not only to complete labs, but to understand common misconfiguration patterns and turn them into reusable AppSec review knowledge.

## Completed Practice

### Lab 1: TryHackMe - OWASP Top 10 2025: Application Design Flaws

**Task:** AS02: Security Misconfigurations
**Platform:** TryHackMe
**Category:** A02:2025 - Security Misconfiguration
**Status:** Completed
**Main pattern:** Invalid input triggering verbose debug and traceback disclosure

External link:

- https://tryhackme.com/room/owasptopten2025two

#### What I Practised

I tested a User Management API that exposed an endpoint similar to:

```http
GET /api/user/<user_id>
```

The page stated that the user ID must be numeric.

When numeric values were supplied, the API returned the same dummy user-like response:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

The important behaviour appeared when a non-numeric value was supplied, such as:

```http
GET /api/user/admin
```

Instead of returning a safe generic error, the API returned verbose debug information.

The response exposed:

- debug information,
- a flag value,
- an internal traceback,
- backend file path,
- function name,
- line number,
- exception type.

The sensitive value is intentionally redacted in my notes:

```text
<redacted-lab-flag>
```

#### What Was Vulnerable

The application failed to handle invalid input safely.

The issue was not that user IDs could be enumerated. The more important issue was that invalid input triggered a verbose debug/error response that exposed internal implementation details and sensitive data.

This is Security Misconfiguration because the application returned developer/debug information to the client.

#### Why It Matters

Verbose errors can help an attacker understand:

- backend language,
- file paths,
- function names,
- exception types,
- application structure,
- possible input validation weaknesses,
- where to focus later testing.

In real applications, similar behaviour could expose:

- API keys,
- environment variables,
- database connection strings,
- framework details,
- stack traces,
- internal file paths,
- secret values.

#### Secure Behaviour

For invalid input such as:

```http
GET /api/user/admin
```

the API should return a generic response such as:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid user ID"
}
```

The detailed traceback should be logged server-side only.

---

### Lab 2: PortSwigger - Information Disclosure on Debug Page

**Platform:** PortSwigger Web Security Academy
**Lab:** Information disclosure on debug page
**Category:** Information Disclosure / Security Misconfiguration
**Status:** Completed
**Main pattern:** Public debug endpoint exposing server configuration and a secret value

External link:

- https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-on-debug-page

#### What I Practised

I reviewed the application normally and inspected the page source.

Inside the HTML source, I found a commented-out debug link:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

Although this link was not visible in the UI, it was still delivered to the browser inside the HTML source.

I then visited the endpoint directly:

```http
GET /cgi-bin/phpinfo.php
```

The endpoint returned a public `phpinfo()` page.

#### What Was Exposed

The `phpinfo()` page disclosed detailed server and PHP configuration information, including:

- PHP version,
- operating system details,
- Server API,
- PHP configuration file path,
- loaded `php.ini` path,
- additional `.ini` files,
- registered PHP streams,
- loaded modules/extensions,
- PHP settings,
- environment/configuration data,
- a sensitive `SECRET_KEY`.

The real `SECRET_KEY` value is intentionally not stored in this repository:

```text
<redacted-secret-key>
```

#### Why It Was Vulnerable

The real vulnerability was not just the HTML comment.

The real issue was that the debug endpoint was publicly accessible.

The comment only helped with discovery. The endpoint itself should not have existed publicly in a production-like environment.

This is Security Misconfiguration because development/debug functionality was deployed or left accessible in a public environment.

#### Why Hiding the Link Was Not Enough

Hiding a link in an HTML comment is not a security control.

Anything sent to the browser should be treated as visible to the user, including:

- HTML comments,
- hidden elements,
- JavaScript files,
- source maps,
- client-side routes,
- API URLs.

A user can discover these using:

- View Source,
- DevTools,
- Burp Suite,
- browser cache,
- crawling,
- content discovery tools.

#### Secure Behaviour

In production, `/cgi-bin/phpinfo.php` should not be publicly accessible.

The best secure outcome is:

```http
HTTP/1.1 404 Not Found
```

because the debug file should not be deployed at all.

If diagnostic functionality is genuinely required, it should be restricted to trusted internal/admin access only. In that case, an unauthorised user should receive:

```http
HTTP/1.1 403 Forbidden
```

or an authentication challenge depending on the application design.

#### Important Remediation Note

Because a secret was exposed, the remediation is not only to remove the page.

The exposed secret should be rotated.

## Key Takeaways

The two completed A02 patterns were:

1. invalid input causing verbose debug/traceback disclosure,
2. a public debug endpoint exposing `phpinfo()` and a sensitive application secret.

The main lesson is that production systems should fail safely and should not expose debug information, diagnostic pages, secrets, environment details, or internal configuration to users.

Detailed review guidance is kept in:

- [A02 overview](01-overview.md)
- [A02 learning notes](05-learning-notes.md)
- [A02 checklist](03-checklist.md)
- [A02 regression tests](04-regression-tests.md)
- [Verbose API error finding](security-findings/01-example-finding.md)
- [Public debug endpoint finding](security-findings/02-public-debug-endpoint-phpinfo.md)

## Related Notes

- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite Proxy and Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
