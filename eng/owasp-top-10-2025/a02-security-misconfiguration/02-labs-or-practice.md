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

## What I Learned

### 1. Security Misconfiguration Often Appears as Information Disclosure

A02 is not always about a complex exploit. It is often about something that should not be visible but is visible because of unsafe configuration.

Examples from these labs:

- verbose error output,
- traceback disclosure,
- public debug endpoint,
- `phpinfo()` exposure,
- environment/configuration leak,
- exposed application secret.

### 2. Invalid Input Is Useful for Testing Error Handling

When an application says:

```text
User ID must be numeric
```

it is useful to test what happens with non-numeric input.

The goal is not always SQL Injection or IDOR. Sometimes the important result is how the application fails.

Useful test values:

```text
admin
abc
test
-1
0
1.5
999999999999999999999
'
%27
```

### 3. HTML Comments Can Leak Useful Recon Information

HTML comments may reveal:

- hidden endpoints,
- old functionality,
- debug tools,
- internal notes,
- temporary links,
- development-only routes.

A comment does not secure anything. If it is sent to the browser, it is visible.

### 4. Debug Pages Are Dangerous in Public Environments

Debug pages such as `phpinfo()` can expose a large amount of sensitive technical information.

Even if some settings look safe, such as `display_errors=Off`, the page itself may still disclose environment details, paths, modules, and secrets.

### 5. Secrets Must Not Be Stored in Error Messages or Exposed Debug Output

If a secret is exposed, the fix must include:

- removing the exposure,
- rotating the secret,
- checking where else it was used,
- reviewing logs and deployment artifacts,
- adding secret scanning.

## What Confused Me or Needed Clarification

### 1. `400` vs `404` vs `403`

For invalid input to an existing API endpoint, `400 Bad Request` is usually appropriate.

Example:

```http
GET /api/user/admin
```

The endpoint exists, but the input is invalid.

For a debug page that should not exist publicly, `404 Not Found` is usually better.

Example:

```http
GET /cgi-bin/phpinfo.php
```

If the diagnostic endpoint intentionally exists but only authorised users should access it, then `403 Forbidden` is appropriate for authenticated users without permission.

### 2. The Comment Was Not the Vulnerability by Itself

The HTML comment was a discovery clue.

The real vulnerability was the public debug endpoint and the exposed configuration/secret.

### 3. Removing the Comment Is Not Enough

Removing the HTML comment reduces discoverability, but it does not fix the issue if `/cgi-bin/phpinfo.php` still exists.

The endpoint must be removed, blocked, or strictly protected.

### 4. Changing the URL Is Not a Proper Fix

Changing `/cgi-bin/phpinfo.php` to another hidden URL is security by obscurity.

A better fix is to remove the debug file from production or restrict it properly.

## Real-World Review Angle

In a real application, I would look for this A02 pattern in:

- HTML comments,
- JavaScript bundles,
- source maps,
- robots.txt,
- sitemap.xml,
- old test routes,
- admin/debug links,
- `/debug`,
- `/config`,
- `/status`,
- `/health`,
- `/server-status`,
- `/actuator`,
- `/phpinfo.php`,
- `/cgi-bin/phpinfo.php`,
- `.env`,
- `.bak`,
- `.old`,
- log files,
- verbose API errors,
- stack traces,
- response headers,
- deployment artifacts.

I would also review whether debug tools or diagnostic pages are deployed to production.

## Related Notes

- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite Proxy and Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)

## Summary

The two practical A02 patterns I completed were:

1. invalid input causing verbose debug/traceback disclosure,
2. public debug endpoint exposing `phpinfo()` and a sensitive application secret.

The main lesson:

> Production systems should not expose debug information, stack traces, diagnostic pages, secrets, environment details, or internal configuration to users.
