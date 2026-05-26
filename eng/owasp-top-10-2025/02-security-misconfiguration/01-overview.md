# A02 Security Misconfiguration

## Scope

This note maps my practical learning to **OWASP Top 10 2025 A02 Security Misconfiguration**.

It is written from the perspective of a Frontend Engineer moving into AppSec. The focus is not only on solving a lab, but on recognising unsafe production behaviour, reviewing configuration symptoms from the browser/API layer, and describing how the issue should be fixed and regression-tested.

## What this category means

Security Misconfiguration happens when an application, API, framework, server, container, cloud service, CMS, reverse proxy or deployment environment is configured in an unsafe way.

The application code may appear to work, but the surrounding configuration or runtime behaviour can still expose:

- sensitive information,
- unnecessary functionality,
- weak defaults,
- debug features,
- unsafe error handling,
- insecure platform behaviour.

In simple terms:

> The application is vulnerable because something around it has been left too open, too verbose, too permissive or too close to a development setup.

## Why it matters

Security Misconfiguration often gives attackers information or access they should not have.

Misconfigurations can expose:

- debug pages,
- stack traces,
- internal file paths,
- framework or server versions,
- environment variables,
- API keys or tokens,
- backup files,
- development endpoints,
- default credentials,
- overly permissive CORS settings,
- unnecessary services or routes,
- missing hardening headers.

For an attacker, this reduces guesswork. A verbose error or exposed debug page can reveal how the application is built, where files are stored, which framework is used, and which parts of the application may be worth attacking next.

## Abuse case

A user sends unexpected input to a public API endpoint.

Instead of returning a controlled client-facing error, the application returns debug information including a traceback, internal file path, function name, line number and a sensitive value.

The attacker uses this information for reconnaissance and to plan follow-on attacks against the application stack.

## Practical example from this sprint

The practice task exposed a User Management API:

```http
GET /api/user/<user_id>
GET /api/user/123
```

The API documentation stated that the user ID must be numeric.

A normal numeric value returned a generic user object:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

However, when a non-numeric value such as `admin` was supplied, the API returned a verbose debug object instead of a controlled error. The response exposed a traceback, internal file path, function name, line number, exception type and a sensitive lab value.

This is Security Misconfiguration because internal debug information and sensitive data appeared in a client-facing error response. The full lab evidence is kept in [02-labs-or-practice.md](02-labs-or-practice.md), and the report-style evidence is kept in [security-findings/01-example-finding.md](security-findings/01-example-finding.md).

## Common examples

Examples of Security Misconfiguration include:

- debug mode enabled in production,
- verbose errors returned to users,
- stack traces exposed in API responses,
- default accounts or passwords left enabled,
- unnecessary routes, services, plugins or admin pages exposed,
- test or staging endpoints reachable from the internet,
- missing or weak security headers,
- overly permissive CORS configuration,
- directory listing enabled,
- exposed configuration files such as `.env`, backups, logs or old deployment files,
- public source maps without an accepted reason,
- cloud storage buckets or services with excessive permissions,
- inconsistent configuration between environments.

## Common root causes

Common root causes include:

- development settings accidentally deployed to production,
- missing global error handling,
- exception messages containing sensitive data,
- secrets included in error messages or logs,
- lack of environment-specific configuration,
- no production hardening checklist,
- no automated checks for security headers or exposed files,
- default framework behaviour left unchanged,
- unclear ownership of security configuration,
- configuration drift between environments.

## Impact

The impact depends on what is exposed.

In this lab, the application exposed a flag and internal traceback information. In a real application, similar behaviour could expose:

- API keys,
- JWT signing secrets,
- database connection strings,
- usernames or emails,
- internal paths,
- framework details,
- source code locations,
- SQL queries,
- stack traces,
- cloud metadata,
- service names,
- internal architecture details.

This information can support further attacks such as path traversal, LFI, injection, SSRF, credential attacks or targeted exploitation of a known framework/component.

## Related internal notes

This category connects with several topics I have already studied:

- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md) - useful for recognising unsafe status codes, headers and response bodies.
- [Burp Suite Proxy and Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md) - useful for comparing normal and error responses.
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md) - useful for finding exposed debug, config, backup or admin paths.
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md) - related when exposed paths or files reveal the filesystem.
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md) - related when configuration exposes internal services or metadata endpoints.
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md) - related when server/web-server configuration allows uploaded files to execute or be served unsafely.

## My takeaway as a Frontend Engineer

As a Frontend Engineer, I may not always configure the backend, server or infrastructure directly, but I can still identify misconfiguration symptoms from the browser, DevTools, API responses and Burp Suite.

Important frontend/AppSec observations:

- API responses should not expose stack traces or internal file paths.
- UI error messages should be user-friendly, not developer-debug output.
- Frontend applications should not rely on secrets embedded in client-side code.
- Production builds should not expose source maps, debug flags or development-only features unless there is a clear accepted reason.
- Client-side error handling should not display raw backend exceptions directly.
- Security headers and CORS behaviour can often be reviewed from the browser.
- Debug endpoints and internal APIs should not be discoverable from public frontend assets.

The key lesson:

> Production systems should fail safely. Invalid input should produce a controlled, generic response, while detailed debugging information should stay in server-side logs.

## What good looks like

A safer implementation should:

- disable debug mode in production,
- use production-specific configuration,
- validate input before business logic runs,
- return generic client-facing error messages,
- log detailed technical errors server-side only,
- keep secrets out of exception messages and response bodies,
- standardise API error response formats,
- remove unnecessary services, routes, plugins and test pages,
- protect admin, debug, config and status endpoints,
- harden headers and CORS behaviour,
- add regression tests that fail if debug information appears in responses.

## External references

- OWASP Top 10 2025 A02 Security Misconfiguration: https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/
- OWASP Cheat Sheet Series: Error Handling: https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
- OWASP Cheat Sheet Series: HTTP Headers: https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html
