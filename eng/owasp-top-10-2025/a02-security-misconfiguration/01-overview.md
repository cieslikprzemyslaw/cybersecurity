# A02:2025 - Security Misconfiguration

## What This OWASP Category Means

Security Misconfiguration happens when an application, API, server, framework, CMS, cloud service, reverse proxy, deployment process, or environment is configured in an unsafe way.

The application code may look correct, but the system can still be vulnerable because something around it has been left exposed, enabled, too permissive, too verbose, or not hardened for production.

In simple terms:

> Security Misconfiguration is when the application or its environment is set up in a way that exposes information, functionality, files, debug tools, or behaviour that should not be publicly available.

## Why It Matters

Modern applications depend on many layers:

- frontend application,
- backend APIs,
- web server,
- application server,
- reverse proxy,
- CMS,
- framework settings,
- environment variables,
- secrets management,
- build process,
- deployment process,
- cloud services,
- logging and monitoring,
- third-party tooling.

A single unsafe setting can expose information that helps an attacker understand the application, target the stack more accurately, or chain the issue with another vulnerability.

Security Misconfiguration often feels less dramatic than SQL Injection or Broken Access Control, but it can be very serious because it may reveal:

- secrets,
- internal paths,
- stack traces,
- framework details,
- server versions,
- environment variables,
- debug endpoints,
- test functionality,
- backup files,
- configuration files,
- sensitive error messages.

## Practical Patterns From My Labs

During this topic, I practised two common A02 patterns.

### Pattern 1: Verbose Error / Debug Response

A TryHackMe lab exposed detailed debug information when invalid input was provided to an API endpoint.

The application expected a numeric user ID:

```http
GET /api/user/<user_id>
```

When a non-numeric value such as `admin` was supplied, the API returned a verbose error response containing:

- internal traceback,
- backend file path,
- function name,
- line number,
- exception type,
- sensitive flag value.

This showed that the application handled invalid input unsafely and returned developer/debug information to the client.

Secure behaviour should return a generic client-facing error such as:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid user ID"
}
```

Detailed traceback information should be logged server-side only.

### Pattern 2: Exposed Debug Page / `phpinfo()`

A PortSwigger lab exposed a hidden debug endpoint through an HTML comment:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

The link was not visible in the UI, but it was still present in the HTML source. Visiting the endpoint directly exposed a `phpinfo()` page.

The page disclosed detailed server and PHP configuration information, including:

- PHP version,
- operating system details,
- Server API,
- loaded configuration file paths,
- additional `.ini` files,
- PHP modules/extensions,
- registered streams,
- PHP settings,
- environment/configuration data,
- a sensitive `SECRET_KEY`.

This showed that a development/debug endpoint was left publicly accessible.

Secure behaviour should be:

- the debug file is not deployed to production,
- the endpoint returns `404 Not Found`,
- or, if diagnostic access is genuinely required, it is restricted to trusted/internal/admin access and returns `403 Forbidden` for unauthorised users.

## Common Examples

Security Misconfiguration can include:

- debug mode enabled in production,
- verbose stack traces shown to users,
- exposed debug endpoints such as `phpinfo()`,
- public `/debug`, `/config`, `/status`, `/server-status`, `/actuator`, or `/health` endpoints,
- exposed `.env`, backup files, logs, source maps, or configuration files,
- directory listing enabled,
- default accounts or default passwords,
- unnecessary services or features enabled,
- weak CORS configuration,
- missing or weak security headers,
- outdated framework/server defaults,
- sensitive information in HTML comments or JavaScript,
- test/admin functionality deployed to production,
- secrets included in exception messages,
- unsafe differences between development, staging, and production environments.

## Common Root Causes

Typical causes include:

- development settings used in production,
- debug files deployed accidentally,
- weak environment separation,
- missing hardening checklist,
- no deployment review for sensitive files,
- no secret scanning,
- no automated checks for debug endpoints,
- framework/server defaults not reviewed,
- verbose error handling left enabled,
- lack of centralised error handling,
- sensitive values included in exception messages,
- assuming hidden frontend links are secure,
- relying on obscurity instead of removal or server-side protection.

## Impact

The impact depends on what is exposed.

Possible impacts include:

- sensitive data disclosure,
- application secret leakage,
- environment variable leakage,
- internal path disclosure,
- framework and version fingerprinting,
- improved attacker reconnaissance,
- targeted exploitation against known versions,
- easier chaining with other vulnerabilities,
- session/token manipulation if secrets are exposed,
- disclosure of implementation details,
- loss of confidence in production hardening.

If a `SECRET_KEY` or similar application secret is exposed, the risk can be high. Depending on how the application uses the secret, an attacker may be able to forge or manipulate trusted values such as signed cookies, session data, CSRF tokens, JWTs, reset tokens, or other protected application data.

The exact impact depends on the application, but exposed secrets should always be treated seriously and rotated.

## Related Vulnerabilities I Have Already Practised

Related topics from my previous learning:

- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Broken Access Control / IDOR](../../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)

These topics are related because misconfiguration often increases the impact of other vulnerabilities. For example, exposed stack traces can help with path traversal testing, exposed framework versions can help with targeted vulnerability research, and exposed secrets may affect authentication/session security.

## My Takeaway as a Frontend Engineer

Frontend code can support security, but it cannot be the security boundary.

Anything sent to the browser should be treated as visible to the user, including:

- HTML comments,
- hidden links,
- JavaScript files,
- source maps,
- client-side routes,
- API URLs,
- feature flags,
- debug references.

Removing a link from the UI or hiding it inside a comment does not protect the backend endpoint.

The real fix must happen through production-safe configuration, backend/server controls, deployment hardening, secret management, and access control where appropriate.

As a Frontend Engineer moving into AppSec, this category is important because many misconfiguration issues are visible from the browser, DevTools, Burp Suite, response headers, page source, JavaScript bundles, or public endpoints.

## What Good Looks Like

A safer application should:

- disable debug mode in production,
- avoid returning stack traces to users,
- use generic client-facing error messages,
- log detailed errors server-side only,
- never include secrets in error messages,
- remove debug files from production deployments,
- block public access to diagnostic endpoints,
- rotate exposed secrets immediately,
- run secret scanning in CI/CD,
- review public assets before deployment,
- harden server/framework defaults,
- apply security headers consistently,
- separate development, staging, and production configuration,
- verify that sensitive endpoints are not discoverable or publicly accessible.

## Summary

Security Misconfiguration is not always a complex exploit. Sometimes the issue is simply that the application exposes something it should not.

The two key patterns I practised were:

1. invalid input causing verbose debug output and traceback disclosure,
2. a hidden debug endpoint exposing `phpinfo()` and a sensitive application secret.

The main lesson:

> Production applications should fail safely, expose minimal information, and never make development/debug functionality publicly accessible.
