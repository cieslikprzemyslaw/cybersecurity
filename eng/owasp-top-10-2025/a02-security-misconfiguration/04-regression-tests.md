# A02:2025 - Security Misconfiguration Regression Tests

## Purpose

Regression tests for Security Misconfiguration should verify that unsafe configuration does not return after a fix.

The goal is to ensure that production-like environments do not expose:

- verbose errors,
- stack traces,
- internal paths,
- debug pages,
- server configuration,
- environment variables,
- secrets,
- development-only routes,
- unsafe headers,
- public diagnostic endpoints.

## Test Area 1: Verbose Error Handling

### Negative Test: Invalid User ID Does Not Expose Debug Details

**Given** the API expects a numeric user ID
**When** a request is sent with a non-numeric value:

```http
GET /api/user/admin
```

**Then** the response should return:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json
```

with a generic message such as:

```json
{
  "error": "Invalid user ID"
}
```

### The Response Must Not Contain

```text
Traceback
Stack trace
Exception
ValueError
TypeError
/app/
app.py
line 21
SECRET_KEY
DATABASE_URL
API_KEY
JWT_SECRET
.env
```

### State Verification

The test should also verify that:

- no sensitive data is returned,
- no internal path is returned,
- no framework-specific debug output is returned,
- the API still logs the detailed error server-side if needed.

## Test Area 2: Valid Input Still Works

### Positive Test: Numeric User ID Still Returns Expected Data

**Given** a valid numeric user ID
**When** the request is sent:

```http
GET /api/user/123
```

**Then** the API should return the expected successful response.

This ensures that the fix does not break normal functionality.

## Test Area 3: Public Debug Endpoint Is Not Accessible

### Negative Test: `phpinfo()` Debug Endpoint Is Removed or Blocked

**Given** the application is running in a production-like environment
**When** a request is sent to:

```http
GET /cgi-bin/phpinfo.php
```

**Then** the response should be one of the following, depending on the intended architecture:

```http
HTTP/1.1 404 Not Found
```

or:

```http
HTTP/1.1 403 Forbidden
```

The preferred production behaviour is `404 Not Found` because the debug file should not be deployed at all.

### The Response Must Not Contain

```text
phpinfo()
PHP Version
System
Server API
Configuration File
Loaded Configuration File
SECRET_KEY
Environment
$_SERVER
$_ENV
allow_url_fopen
allow_url_include
```

## Test Area 4: Debug Page Is Not Discoverable From HTML

### Negative Test: Page Source Does Not Expose Debug Links

**Given** a public page is loaded
**When** the HTML source is inspected
**Then** it should not contain debug links such as:

```text
/cgi-bin/phpinfo.php
/phpinfo.php
/debug
/config
/server-status
/actuator
```

### Important Note

This test reduces accidental disclosure, but it is not enough by itself.

Even if the link is removed from the HTML, the backend endpoint must still be removed or blocked.

## Test Area 5: Secret Exposure

### Negative Test: Responses Do Not Contain Secrets

For public unauthenticated and normal authenticated requests, responses should not contain:

```text
SECRET_KEY
JWT_SECRET
SESSION_SECRET
DATABASE_URL
DB_PASSWORD
API_KEY
PRIVATE_KEY
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

### Additional Verification

If a secret was previously exposed:

- rotate the exposed secret,
- verify the old secret no longer works,
- verify the new secret is not exposed,
- review logs and artifacts,
- run secret scanning against the repository and build output.

## Test Area 6: Known Debug Files Are Not Deployed

### Build/CI Test

The deployment pipeline should fail if production artifacts contain files such as:

```text
phpinfo.php
debug.php
test.php
info.php
.env
.env.local
.env.production
backup.zip
backup.tar.gz
database.sql
dump.sql
```

### Example Assertion

```text
No known debug or secret-bearing files should exist in the production build artifact.
```

## Test Area 7: Content Discovery Smoke Test

### Manual or Automated Verification

Run a small content discovery check against production-like environments for known risky paths:

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
backup.zip
error.log
```

### Expected Result

Unexpected sensitive paths should return:

```http
404 Not Found
```

or:

```http
403 Forbidden
```

depending on whether the endpoint should exist and be protected.

## Test Area 8: Security Header Review

### Header Checks

Verify that security-relevant headers are present and appropriate for the application:

```text
Content-Security-Policy
X-Frame-Options
Strict-Transport-Security
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
Set-Cookie
Access-Control-Allow-Origin
Access-Control-Allow-Credentials
```

### Version Leakage Checks

Review whether these headers expose unnecessary implementation details:

```text
Server
X-Powered-By
```

The goal is not always to remove every technology hint, but to avoid unnecessary version/detail leakage.

## Test Area 9: CORS Configuration

### Negative Test: Sensitive APIs Do Not Use Overly Broad CORS

For sensitive authenticated endpoints:

- `Access-Control-Allow-Origin` should not be blindly set to `*`.
- credentialed requests should only allow trusted origins.
- `Access-Control-Allow-Credentials: true` must be reviewed carefully.
- origins should be validated safely.

## Test Area 10: Error Consistency

### Negative Test: Different Error Paths Do Not Leak Debug Details

Test multiple invalid inputs:

```text
abc
admin
-1
0
1.5
999999999999999999999
'
%27
null
undefined
```

Expected behaviour:

- generic client-facing error,
- no traceback,
- no internal path,
- no secrets,
- no database/framework details.

## Manual Verification Checklist

Before closing an A02 fix, manually verify:

- invalid input returns safe generic errors,
- debug pages are removed or blocked,
- HTML source does not reveal debug links,
- JavaScript bundles do not expose secrets or internal routes,
- source maps are handled intentionally,
- `.env` and config files are not public,
- response headers do not leak unnecessary details,
- secret scanning has been run,
- exposed secrets have been rotated,
- server-side logs still contain enough detail for debugging.

## What Should Fail After the Fix

After remediation, these should fail:

```text
GET /api/user/admin exposing traceback
GET /cgi-bin/phpinfo.php exposing phpinfo()
Finding SECRET_KEY in public responses
Finding internal file paths in client-facing errors
Finding debug URLs in public HTML comments
Accessing development-only diagnostic pages publicly
```

## What Should Continue to Work After the Fix

These should still work:

```text
Valid API requests
Normal user-facing error handling
Server-side logging
Admin-only diagnostics if intentionally supported and properly protected
Monitoring and health checks if designed safely
```

## Acceptance Criteria

A fix can be accepted when:

- invalid API input returns a generic safe error,
- public responses do not contain stack traces,
- public responses do not contain internal paths,
- public responses do not contain secrets,
- debug pages are removed or blocked,
- exposed secrets have been rotated,
- production artifacts do not include known debug files,
- CI/CD includes checks to reduce recurrence,
- normal functionality still works.

## Summary

Security Misconfiguration regression tests should check not only whether a specific lab payload is fixed, but whether the application consistently avoids exposing internal details.

The key checks are:

> no verbose errors, no public debug pages, no exposed secrets, no unsafe production configuration.
