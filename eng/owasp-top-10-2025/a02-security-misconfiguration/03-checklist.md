# A02 Checklist: Security Misconfiguration

Use this checklist during code review, API review, manual testing, frontend deployment review or AppSec review.

The main question is:

> Does the application expose unsafe behaviour because configuration, error handling or platform hardening is too open, too verbose or too close to development mode?

## 1. Understand the surface

Before testing, identify:

- What application, API or service is exposed?
- Is this production, staging, preview or a local lab?
- Which framework, server, CMS, container or cloud service is involved?
- Which endpoints are public?
- Which endpoints should be internal only?
- What error format does the application use?
- Are there admin, debug, status, health or config endpoints?
- Are frontend bundles, source maps or static assets exposed?

Examples of sensitive configuration areas:

- error handling,
- debug mode,
- security headers,
- CORS,
- cookies,
- source maps,
- admin routes,
- cloud storage,
- default accounts,
- exposed files,
- unnecessary services or plugins.

## 2. Error handling and debug output

Check:

- Are debug or development settings disabled in production?
- Are error responses generic and safe for users?
- Are technical details logged server-side only?
- Does the API return stack traces?
- Does the API reveal internal file paths?
- Does the API reveal function names, class names, line numbers or framework details?
- Does the API expose exception types such as `ValueError`, `TypeError`, `NullReferenceException` or database errors?
- Does the API expose SQL queries, schema names, table names or connection details?
- Does the API expose secrets, tokens, flags, API keys or environment variables?
- Does the API return consistent and safe error formats?

## 3. Code review questions

Ask:

- Where is production configuration defined?
- Is there a clear separation between development, staging and production settings?
- Is debug mode controlled by an environment variable or deployment profile?
- Is there a global error handler?
- Are exceptions converted into generic client-facing responses?
- Are secrets ever included in exception messages?
- Are raw backend errors passed directly to the frontend?
- Are default framework settings reviewed before deployment?
- Are unnecessary routes, plugins or services disabled?
- Are hardening checks part of deployment or CI?

## 4. Testing questions

For each important endpoint, test:

- What happens with valid input?
- What happens when required parameters are missing?
- What happens when parameters use the wrong type?
- What happens when IDs are non-numeric, too long, negative, decimal or malformed?
- What happens with special characters?
- What happens with unexpected HTTP methods?
- What happens when common debug/config paths are requested?
- Does the response reveal internal details?
- Does the response expose sensitive values?
- Does the response remain safe across `400`, `401`, `403`, `404` and `500` cases?

Example invalid input tests:

```http
GET /api/user/admin
GET /api/user/abc
GET /api/user/-1
GET /api/user/1.5
GET /api/user/%27
GET /api/user/999999999999999999999999
```

Look for:

- `Traceback`,
- `Exception`,
- `ValueError`,
- `TypeError`,
- file paths such as `/app/...`, `/var/www/...`, `C:\...`,
- framework names,
- stack traces,
- database errors,
- secrets,
- tokens,
- API keys,
- environment variable names or values.

## 5. Frontend-specific checks

Frontend review can reveal misconfiguration symptoms quickly.

Review:

- Are source maps exposed in production without a clear reason?
- Are debug flags or development messages visible in the console?
- Are backend errors displayed directly in the UI?
- Are API errors rendered raw to the user?
- Are sensitive values present in JavaScript bundles?
- Are internal endpoints listed in frontend assets?
- Are staging, preview or internal URLs exposed in the client bundle?
- Are comments in HTML or JavaScript revealing hidden routes, credentials, tickets or internal notes?
- Does the frontend handle `400`, `401`, `403` and `500` responses safely?

But also confirm:

- The backend response is safe even without frontend filtering.
- Raw backend exceptions are not part of the public API contract.
- Secrets are not embedded in client-side code.

## 6. Header and platform checks

Check response headers and platform behaviour:

- Are security headers present and appropriate?
- Is `Content-Security-Policy` configured where practical?
- Is `X-Content-Type-Options` present?
- Is `Referrer-Policy` present?
- Is `Permissions-Policy` present where useful?
- Is `Strict-Transport-Security` enabled for HTTPS production sites?
- Is clickjacking protection configured with `frame-ancestors` or `X-Frame-Options`?
- Are server/framework versions exposed unnecessarily?
- Is CORS configured safely?
- Are cookies configured with `HttpOnly`, `Secure` and `SameSite` where appropriate?
- Is directory listing disabled?
- Are backup, log, config or temporary files blocked from public access?
- Are admin/debug/status endpoints protected?

## 7. Developer red flags

Red flags include:

- `debug=True` in production,
- development error pages exposed publicly,
- exception messages returned directly to API clients,
- `res.send(error)` or equivalent patterns,
- raw backend error objects passed to the frontend,
- secrets embedded in exception messages,
- environment variables printed in responses,
- overly permissive CORS such as reflecting arbitrary origins,
- public `.env`, `.git`, backup, log or config files,
- unprotected `/debug`, `/config`, `/admin`, `/status`, `/health`, `/actuator` or `/server-status` endpoints,
- missing environment separation,
- "It is only staging, so it does not need hardening."

## 8. Remediation checklist

A safer implementation should:

- disable debug mode in production,
- add a global error handler,
- return generic client-facing error messages,
- log detailed errors server-side only,
- remove secrets from exception messages,
- validate input before business logic runs,
- standardise API error responses,
- separate production configuration from development configuration,
- block access to debug/config/internal endpoints,
- disable directory listing,
- remove default credentials,
- remove unnecessary services, plugins, routes and test pages,
- harden security headers,
- review CORS settings,
- add automated regression tests for error disclosure.

## 9. Safe implementation notes

For invalid input, prefer a controlled response such as:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid user ID"
}
```

Do not return:

```text
Traceback
File "/app/app.py"
ValueError
DATABASE_URL
API_KEY
JWT_SECRET
```

Detailed information should go to server-side logs, monitoring or error tracking tools, not to the user.

## 10. Review outcome

A feature should pass the A02 review only if:

- production debug behaviour is disabled,
- invalid input returns controlled errors,
- client-facing responses do not expose stack traces, paths, exception types or secrets,
- unnecessary endpoints and files are not publicly reachable,
- headers, CORS and cookies are configured intentionally,
- frontend code does not expose secrets or raw debug output,
- regression tests exist for the observed misconfiguration class.
