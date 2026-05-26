# A02 Labs and Practice: Security Misconfiguration

## Completed practice

### TryHackMe

**Room/module:** OWASP Top 10 2025: Application Design Flaws  
**Task:** Task 2 - AS02: Security Misconfigurations  
**Status:** Completed  
**Main pattern:** Verbose API error response exposing debug information and sensitive data

## Lab context

This was an intentionally vulnerable learning task used for legal practice and documentation.

The lab presented a User Management API with the following documented endpoint:

```http
GET /api/user/<user_id>
```

Example:

```http
GET /api/user/123
```

The page stated that the user ID must be numeric.

This was a good A02 exercise because the issue was not found by bypassing authentication or changing ownership checks. The issue appeared when the application received unexpected input and returned unsafe debug information.

## What I practised

In this lab, I practised:

- reading API documentation exposed by the application,
- testing expected input first,
- testing invalid input after understanding the expected format,
- comparing normal responses with error responses,
- identifying verbose error output,
- recognising traceback disclosure,
- distinguishing Security Misconfiguration from IDOR or Broken Access Control,
- thinking about safe production error handling,
- turning lab evidence into an AppSec-style finding.

## Important distinction

The issue was not that the user could access another user's record by changing an ID.

The issue was that invalid input caused the application to expose debug details and a sensitive value in the API response.

This makes the vulnerability a Security Misconfiguration issue, not primarily an IDOR or Broken Access Control issue.

## Observed behaviour

### Normal numeric request

A numeric user ID returned a generic user object:

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

Changing numeric IDs returned similar dummy data. This suggested that simple ID enumeration was not the main issue in this task.

### Invalid non-numeric request

A non-numeric value such as `admin` triggered a verbose error response:

```http
GET /api/user/admin
```

The response included debug information similar to:

```json
{
  "debug_info": {
    "flag": "<redacted-lab-flag>",
    "error": "Invalid user ID format: admin. Flag: <redacted-lab-flag>",
    "traceback": "Traceback (most recent call last):\n  File \"/app/app.py\", line 21, in get_user\n    raise ValueError(...)\nValueError: Invalid user ID format: admin. Flag: <redacted-lab-flag>\n"
  }
}
```

The real lab flag was intentionally redacted from these notes. Secrets, flags, tokens and real sensitive values should not be committed to the repository.

## What was vulnerable

The vulnerable behaviour was the error response returned for invalid input:

```http
GET /api/user/admin
```

The response exposed:

- an internal file path: `/app/app.py`,
- a function name: `get_user`,
- a line number: `line 21`,
- an exception type: `ValueError`,
- traceback details,
- a sensitive lab flag value.

## Why this is Security Misconfiguration

This is Security Misconfiguration because the application exposed verbose debug information and sensitive data through a client-facing API error.

It is not mainly IDOR because the issue was not about accessing another user's object by changing an ID.

It is not mainly Broken Access Control because the issue was not about missing authorization checks.

The root problem was unsafe production-style error handling and debug information disclosure.

## What confused me or was worth noticing

### Numeric IDs returned the same user

At first, changing numeric IDs seemed like the obvious test because the endpoint was `/api/user/<user_id>`. However, every numeric ID returned the same dummy user.

That was a clue that the task was probably not about IDOR or Broken Access Control.

### The phrase "User ID must be numeric" was a hint

The important test was not only trying different numeric IDs. The better A02 test was:

> What happens when the application receives input that breaks the expected format?

Supplying a non-numeric value triggered the verbose error response.

### 400 vs 404

For a request like:

```http
GET /api/user/admin
```

a safe response should normally be:

```http
400 Bad Request
```

because the endpoint exists, but the input format is invalid.

A `404 Not Found` may be used in some designs, but for input validation errors `400 Bad Request` is usually clearer.

## How I would test this in a real app

I would look for this pattern in:

- public API endpoints,
- user lookup routes,
- search and filter parameters,
- admin and status pages,
- health or debug endpoints,
- upload processing,
- JSON APIs,
- CMS-backed routes,
- reverse-proxy and web-server error pages,
- staging or preview deployments.

For each area, I would test:

- valid input first,
- missing input,
- invalid types,
- very long values,
- special characters,
- unexpected HTTP methods,
- direct access to common debug/config paths,
- whether errors contain tracebacks, file paths, secrets or framework details.

## Review result

The practical lesson from this lab is simple: invalid input is normal, but debug output must not become part of the public API.

From a non-technical point of view, this is like showing a customer the internal repair manual, staff notes and secret storage codes whenever they type an invalid account number.

The main takeaways:

- the endpoint worked for expected numeric input,
- the unsafe behaviour appeared only when the input format was broken,
- the API returned implementation details that belonged in server-side logs,
- sensitive values should never be included in exception messages,
- a good fix needs tests that check both the safe error status and the absence of debug strings.

## Related internal notes

- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite Proxy and Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)

## Future practice ideas

Possible next A02-related practice:

- review a local app for debug mode and verbose error responses,
- check security headers and CORS behaviour on a test deployment,
- perform content discovery for exposed `.env`, backup, log or debug paths in a legal lab,
- compare development vs production configuration in a sample project,
- write regression tests for generic API error handling.
