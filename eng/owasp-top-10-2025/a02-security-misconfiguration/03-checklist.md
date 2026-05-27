# A02:2025 - Security Misconfiguration Checklist

## Purpose

Use this checklist when reviewing applications, APIs, frontend assets, deployment configuration, or environments for Security Misconfiguration issues.

This checklist focuses on practical AppSec review questions from a developer and tester perspective.

## Core Question

Ask:

> Is the application exposing anything that should only be visible to developers, administrators, infrastructure, logs, or trusted internal systems?

## 1. Error Handling Review

### Testing Questions

- Does invalid input return a safe generic error?
- Does the application expose stack traces?
- Does the response include file paths?
- Does the response include function names?
- Does the response include line numbers?
- Does the response include framework errors?
- Does the response include database errors?
- Does the response include exception types such as `ValueError`, `TypeError`, `NullReferenceException`, `SQLException`, or similar?
- Does the response expose secrets, tokens, flags, environment variables, or configuration values?
- Are detailed errors only logged server-side?

### Example Bad Response

```text
Traceback (most recent call last):
  File "/app/app.py", line 21, in get_user
ValueError: Invalid user ID format
```

### Safer Response

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid user ID"
}
```

### Red Flags

- `Traceback`
- `Stack trace`
- `Exception`
- `ValueError`
- `TypeError`
- `SQL syntax`
- `/app/`
- `/var/www/`
- `/home/`
- `C:\`
- `SECRET_KEY`
- `DATABASE_URL`
- `API_KEY`
- `JWT_SECRET`
- `AWS_ACCESS_KEY`
- `DEBUG=true`

## 2. Debug Endpoint Review

### Testing Questions

- Are debug endpoints publicly accessible?
- Are diagnostic pages deployed to production?
- Is `phpinfo()` exposed?
- Are framework debug pages accessible?
- Are `/debug`, `/config`, `/status`, `/health`, `/server-status`, `/actuator`, or similar endpoints exposed?
- Are debug URLs referenced in HTML comments or JavaScript?
- Are old test/admin/development routes still deployed?
- Are debug endpoints protected by authentication and authorization?
- Should the endpoint exist at all in production?

### Common Paths to Check

```text
/debug
/config
/status
/health
/server-status
/actuator
/phpinfo.php
/cgi-bin/phpinfo.php
.env
.env.local
.env.production
config.php
backup.zip
backup.tar.gz
app.log
error.log
```

### Red Flags

- Public `phpinfo()` page.
- Debug endpoint linked in HTML comments.
- Diagnostic output exposed without authentication.
- Server, framework, or environment configuration shown to users.
- Sensitive values such as `SECRET_KEY` exposed.
- Debug endpoint hidden only by URL or UI.

## 3. HTML / Frontend Asset Review

### Testing Questions

- Do HTML comments reveal hidden endpoints?
- Do comments reveal debug URLs?
- Do JavaScript bundles contain internal API URLs?
- Are source maps publicly available?
- Are feature flags exposed client-side?
- Are secrets accidentally placed in frontend code?
- Are development-only routes included in production builds?
- Are staging or internal URLs visible?
- Are comments treated as safe even though they are sent to the browser?

### Important Rule

> Anything sent to the browser should be treated as visible to the user.

This includes:

- HTML,
- CSS,
- JavaScript,
- comments,
- hidden inputs,
- source maps,
- client-side routes,
- bundled configuration.

## 4. Secrets and Environment Variables

### Testing Questions

- Are secrets stored in source code?
- Are secrets exposed in error responses?
- Are secrets exposed in debug pages?
- Are secrets exposed in frontend bundles?
- Are secrets exposed in logs?
- Are secrets exposed through environment dumps?
- Is secret scanning enabled in CI/CD?
- Are exposed secrets rotated?
- Are separate secrets used per environment?

### Red Flags

- `SECRET_KEY`
- `JWT_SECRET`
- `SESSION_SECRET`
- `DATABASE_URL`
- `DB_PASSWORD`
- `API_KEY`
- `PRIVATE_KEY`
- `AWS_SECRET_ACCESS_KEY`
- `.env` files in web root
- secrets inside exception messages

### Remediation Checklist

- Remove exposed secrets.
- Rotate exposed secrets immediately.
- Review where the secret was used.
- Check git history.
- Check logs and artifacts.
- Add secret scanning.
- Use a proper secrets manager where possible.

## 5. Response Header Review

### Testing Questions

- Are security headers present?
- Are server/framework versions exposed unnecessarily?
- Is CORS configured safely?
- Are cookies configured securely?
- Are debug or internal headers exposed?

### Headers to Review

```text
Content-Security-Policy
X-Frame-Options
Strict-Transport-Security
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
Access-Control-Allow-Origin
Access-Control-Allow-Credentials
Set-Cookie
Server
X-Powered-By
```

### Red Flags

- `X-Powered-By` exposing technology unnecessarily.
- `Server` exposing detailed versions.
- `Access-Control-Allow-Origin: *` on sensitive APIs.
- `Access-Control-Allow-Credentials: true` with overly broad origins.
- Cookies missing `HttpOnly`, `Secure`, or appropriate `SameSite`.
- No `Content-Security-Policy` where the application needs browser-side hardening.

## 6. File and Directory Exposure

### Testing Questions

- Is directory listing enabled?
- Are backup files accessible?
- Are log files accessible?
- Are `.env` files accessible?
- Are source maps accessible?
- Are old deployment files accessible?
- Are uploaded files served from a safe location?
- Are config files inside the web root?

### Common Sensitive Files

```text
.env
.env.local
.env.production
config.php
config.json
settings.py
web.config
appsettings.json
backup.zip
backup.tar.gz
database.sql
dump.sql
error.log
access.log
package-lock.json
composer.lock
```

Not every file is equally sensitive, but unexpected public access should be reviewed.

## 7. Production Hardening Questions

- Is debug mode disabled?
- Are verbose errors disabled?
- Are detailed errors logged server-side only?
- Are default credentials removed?
- Are unused routes disabled?
- Are unused services disabled?
- Are unnecessary modules removed?
- Are production and staging configurations separated?
- Are secrets stored outside the codebase?
- Is deployment output reviewed?
- Are test/debug files excluded from production builds?
- Are server defaults hardened?

## 8. Code Review Questions

- Can exception messages include secrets?
- Are errors handled centrally?
- Is input validated before risky logic runs?
- Are framework debug settings environment-specific?
- Are development utilities excluded from production builds?
- Are debug routes registered only in development?
- Are environment variables ever returned to the client?
- Are internal paths exposed in API responses?
- Are sensitive values accidentally logged or returned?

## 9. Testing Questions

- What happens when invalid input is supplied?
- What happens when required parameters are missing?
- What happens when a parameter has the wrong type?
- What happens when the value is very long?
- What happens when special characters are supplied?
- Does the API return safe errors consistently?
- Can hidden endpoints be found through comments, JS, or crawling?
- Are debug/config endpoints publicly reachable?
- Does content discovery find unexpected files?

## 10. Developer Red Flags

Be careful when you see:

- commented-out debug links,
- TODO comments referencing internal routes,
- debug tools in production,
- `phpinfo()` files,
- `.env` files near public folders,
- verbose API errors,
- stack traces in browser responses,
- secrets in exception messages,
- development credentials,
- staging URLs in production JavaScript,
- production builds containing source maps without a reason,
- server/framework versions exposed in headers,
- permissive CORS copied from development.

## 11. Remediation Checklist

For verbose errors:

- Add central/global error handling.
- Return generic client-facing messages.
- Log technical details server-side only.
- Disable debug mode in production.
- Validate input before deeper application logic.
- Remove secrets from exception messages.
- Add regression tests for invalid input.

For exposed debug endpoints:

- Remove debug files from production.
- Block debug paths at the server/reverse proxy level.
- Restrict diagnostic tools to internal/admin access if absolutely required.
- Remove debug references from HTML and frontend bundles.
- Rotate any exposed secrets.
- Review git history and deployment artifacts.
- Add CI/CD checks for known debug files.

For exposed secrets:

- Remove the exposure.
- Rotate the secret.
- Check logs and artifacts.
- Check repository history.
- Add secret scanning.
- Review the impact of the exposed secret.
- Ensure secrets are environment-specific.

## 12. Safe Implementation Notes

Good production behaviour should include:

- minimal error responses,
- server-side logging,
- centralised exception handling,
- environment-specific configuration,
- no public debug pages,
- no secrets in client responses,
- no debug URLs in HTML comments,
- no development-only tools in production,
- controlled diagnostic access,
- secure default configuration,
- automated checks in CI/CD.

## Frontend Engineer Takeaway

Frontend can support security, but frontend is not the security boundary.

Do not rely on:

- hidden links,
- HTML comments,
- disabled buttons,
- hidden inputs,
- client-side routes,
- client-side checks,
- obscured URLs.

If a debug endpoint exists on the server and is publicly reachable, hiding the link in the frontend does not protect it.

The backend, server configuration, deployment pipeline, and environment hardening must prevent sensitive debug functionality from being exposed.
