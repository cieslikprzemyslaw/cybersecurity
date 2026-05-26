# A02 Regression Tests: Security Misconfiguration

## Purpose

Regression tests should prove that the configuration and error-handling fix works and that verbose debug information cannot be reintroduced later.

For Security Misconfiguration, it is not enough to check that invalid input returns an error. Tests should also verify that the error response does not expose internal implementation details or sensitive values.

## Target scenario

The completed practice task showed a User Management API where:

- valid numeric user IDs returned a normal response,
- a non-numeric user ID triggered a verbose debug response,
- the response exposed a traceback, internal file path, function name, line number, exception type and sensitive lab flag.

The regression tests below are designed to prevent this pattern.

## Positive tests

### 1. Valid numeric user ID still works

```gherkin
Given the API is running with production-safe configuration
When a user sends GET /api/user/123
Then the response should be successful
And the response should return the expected user response structure
And the response should not include debug information
```

Expected safe response shape:

```json
{
  "email": "john@example.com",
  "id": "9999",
  "name": "John Doe"
}
```

### 2. Frontend still handles normal API responses

```gherkin
Given the frontend consumes the user API
When the API returns a valid user response
Then the UI should render the expected user state
And no debug-only fields should be required by the frontend
```

## Negative tests

### 3. Non-numeric user ID does not expose debug information

```gherkin
Given the API is running with production-safe configuration
When a user sends GET /api/user/admin
Then the response status should be 400
And the response body should contain a generic error message
And the response body should not contain traceback information
And the response body should not contain internal file paths
And the response body should not contain exception types
And the response body should not contain secrets
```

Example safe response:

```json
{
  "error": "Invalid user ID"
}
```

### 4. Other invalid values are handled safely

Test values:

```text
abc
admin
-1
1.5
999999999999999999999999
%27
null
undefined
```

For each value:

```gherkin
When the invalid value is used as the user ID
Then the API should return a controlled client-facing error
And the response should not expose internal implementation details
```

### 5. Unexpected methods do not expose debug information

```gherkin
Given the user endpoint only supports expected methods
When a user sends an unsupported method to /api/user/123
Then the response should be controlled
And the response should not include raw framework or server error output
```

## Content checks

Automated tests should check that public responses do not contain dangerous strings.

The response should not contain:

```text
Traceback
Exception
ValueError
TypeError
/app/
app.py
line 
debug_info
DATABASE_URL
API_KEY
JWT_SECRET
SECRET_KEY
TOKEN
flag
```

These checks are not a complete security solution, but they are useful regression guards for this specific vulnerability class.

## Behaviour verification

This issue is mainly information disclosure, so there may not be a database state change to verify.

However, tests should still confirm:

- the API remains available,
- valid input still works,
- invalid input returns a controlled error,
- no debug-only fields are returned,
- no sensitive values are present in client-facing responses,
- frontend error rendering remains user-friendly.

## Server-side logging verification

Detailed error information can be useful for developers, but it must stay server-side.

Test idea:

```gherkin
Given a user submits invalid input
When the API handles the error
Then the client receives a generic error
And the server logs contain enough detail for debugging
And the logs are not exposed through the API response
```

The log should be protected and accessible only to authorised operational staff.

## Environment configuration tests

Add checks for environment-specific configuration:

```gherkin
Given the application is deployed to production
Then debug mode should be disabled
And development error pages should not be exposed
And production error handlers should be active
```

Examples of automated checks:

- assert `DEBUG=false` or equivalent,
- assert framework debug pages are disabled,
- assert raw exception handlers are not active,
- assert production configuration is loaded,
- assert test/development endpoints are not publicly reachable.

## Manual verification

Manual verification steps:

1. Send a valid request such as `GET /api/user/123`.
2. Confirm the normal response still works.
3. Send invalid requests such as `GET /api/user/admin`, `GET /api/user/abc` and `GET /api/user/%27`.
4. Confirm the response is a controlled client-facing error.
5. Confirm there is no traceback, file path, function name, line number, exception type, debug object or sensitive value.
6. Confirm the frontend does not render raw backend errors.

## Automation idea

```javascript
test("invalid user ID does not expose debug information", async () => {
  const response = await request(app).get("/api/user/admin");

  expect(response.status).toBe(400);
  expect(response.body.error).toBe("Invalid user ID");

  const body = JSON.stringify(response.body);

  expect(body).not.toContain("Traceback");
  expect(body).not.toContain("/app/app.py");
  expect(body).not.toContain("ValueError");
  expect(body).not.toContain("debug_info");
  expect(body).not.toContain("flag");
  expect(body).not.toContain("SECRET");
  expect(body).not.toContain("TOKEN");
});
```

## Logging and alerting ideas

For repeated invalid requests or attempts to access debug/config paths, consider logging:

- endpoint,
- invalid parameter name and safe summary,
- authenticated user ID if present,
- timestamp,
- source IP or request metadata,
- response status,
- rule or validation failure.

Avoid logging secrets, session cookies or sensitive tokens.

A future A09 review should check whether repeated probing of debug/config paths is visible to defenders.

## Acceptance criteria

The fix is acceptable only when:

- production debug mode is disabled,
- invalid input returns a generic client-facing error,
- public responses do not expose stack traces, internal paths, exception types or secrets,
- detailed errors are available only in protected server-side logs,
- valid API behaviour still works,
- frontend error handling remains user-friendly,
- tests cover both valid and invalid cases,
- tests fail if verbose debug strings reappear.
