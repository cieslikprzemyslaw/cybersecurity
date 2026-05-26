# Example Security Finding: Verbose API Error Response Exposes Debug Information and Sensitive Data

## Summary

The User Management API exposes verbose debug information when a non-numeric user ID is supplied to the `/api/user/<user_id>` endpoint.

The response includes internal implementation details such as a backend file path, function name, line number, exception type, traceback and a sensitive lab flag value.

This behaviour indicates unsafe production-style error handling and maps to OWASP Top 10 2025 A02: Security Misconfiguration.

## Affected area

- Feature: User Management API
- Endpoint: `GET /api/user/<user_id>`
- Affected behaviour: Invalid user ID error handling
- User type: Any user who can reach the endpoint
- OWASP mapping: A02:2025 Security Misconfiguration

## Severity

**Medium**

The issue exposes sensitive technical information and a sensitive lab value. It does not directly provide full system compromise on its own, but the disclosed information can materially help an attacker with reconnaissance and follow-on attacks.

Severity may become High if real secrets, credentials, tokens or exploitable internal paths are exposed.

## Risk / Impact

A user can trigger verbose error output by supplying invalid input.

The response discloses technical details that should not be visible to users, including:

- internal file path,
- backend function name,
- source code line number,
- exception type,
- traceback,
- sensitive value included in the exception message.

In a real application, similar behaviour could expose:

- API keys,
- JWT secrets,
- database connection strings,
- environment variables,
- framework details,
- database errors,
- internal service names,
- internal file paths,
- source code structure.

This information can help an attacker understand the backend implementation and craft more targeted attacks. It may also support exploitation of other vulnerabilities such as path traversal, LFI, injection, SSRF, exposed debug endpoints or insecure file access.

## Root cause

The application returns raw or overly detailed exception information to the client when input validation fails.

The likely root causes are:

- debug-style error handling enabled in a production-like context,
- missing or incomplete global error handler,
- unsafe exception message construction,
- sensitive values included in exception messages,
- lack of generic client-facing error responses,
- input validation errors not handled safely.

## Evidence

### Normal request returns expected response

A numeric request returns a generic user object:

```http
GET /api/user/123
```

Example response:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

### Invalid request exposes debug output

A non-numeric request triggers verbose debug output:

```http
GET /api/user/admin
```

Example response with sensitive values redacted:

```json
{
  "debug_info": {
    "flag": "<redacted-lab-flag>",
    "error": "Invalid user ID format: admin. Flag: <redacted-lab-flag>",
    "traceback": "Traceback (most recent call last):\n  File \"/app/app.py\", line 21, in get_user\n    raise ValueError(...)\nValueError: Invalid user ID format: admin. Flag: <redacted-lab-flag>\n"
  }
}
```

The response reveals:

- `/app/app.py`,
- `get_user`,
- `line 21`,
- `ValueError`,
- traceback details,
- a sensitive lab value.

## Expected secure behaviour

For invalid user ID input, the API should return a controlled generic error response.

Example:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid user ID"
}
```

The response should not include:

- traceback,
- file paths,
- function names,
- line numbers,
- exception types,
- secrets,
- flags,
- tokens,
- environment variables,
- raw backend error objects.

Detailed error information should be logged server-side only.

## Remediation

Implement production-safe error handling for the API.

Recommended fixes:

- Disable debug mode in production.
- Add a global API error handler.
- Validate `user_id` before entering application logic.
- Return generic client-facing error messages for invalid input.
- Log detailed exception information server-side only.
- Remove secrets, flags, tokens and environment values from exception messages.
- Standardise API error response formats.
- Review other endpoints for similar verbose error behaviour.
- Confirm production, staging and development configurations are separated.
- Add automated tests that fail if verbose debug information appears in responses.

## Regression test ideas

Add regression coverage for:

- non-numeric user IDs such as `admin` and `abc`,
- malformed values such as `%27`,
- very long user ID values,
- valid numeric user IDs after the fix,
- frontend rendering of invalid-input errors,
- content checks that fail if `Traceback`, `/app/app.py`, `ValueError`, `debug_info`, `flag`, `SECRET` or `TOKEN` appear in public responses.

Detailed test cases are captured in [04-regression-tests.md](../04-regression-tests.md).

## OWASP mapping

- OWASP Top 10 2025: A02 Security Misconfiguration
- Related future mapping: A10 Mishandling Exceptional Conditions
- Related review area: production-safe error handling and debug information disclosure

## Developer takeaway

Invalid input is normal and expected. A secure application should handle it safely.

The backend should return a clear but generic client-facing error, while detailed debugging information should remain in server-side logs.

Do not let internal exception details become part of the public API contract.
