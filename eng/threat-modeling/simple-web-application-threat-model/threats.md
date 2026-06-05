# Threat Register

All entries are hypothetical and remain open because no real implementation was tested.

## 1. Modified API Request

- **Component:** HTTPS API requests
- **STRIDE category:** Tampering
- **Severity:** High
- **Status:** Open

### Description

A user can modify frontend-generated API requests before they reach the backend, including values such as `userId`, role, price, or object identifiers.

### Validation

Review whether the backend independently identifies the authenticated user, verifies permissions, validates all client-controlled values, and returns only the data and actions the user is authorised to access.

Test modified requests to confirm that changing `userId`, role, price, or object identifiers does not bypass server-side controls.

### Mitigation

Treat every request field as attacker-controlled. Enforce authentication, authorisation, input validation, ownership checks, and business rules on the backend. Return only the data required for the authenticated user and requested operation.

## 2. Broken Object-Level Authorization

- **Component:** Backend API
- **STRIDE category:** Elevation of Privilege
- **Severity:** High
- **Status:** Open

### Description

The backend may allow a user to access or modify another user's object by changing an object identifier in the request.

### Validation

Verify that the backend identifies the authenticated user from a validated session cookie or access token and does not trust a client-supplied `userId`.

Test modified object identifiers while keeping the same authenticated session to confirm that one user cannot access another user's data.

### Mitigation

Verify object ownership and permissions on every request. Do not rely on hidden frontend controls or client-supplied user identifiers.

## 3. Session or Token Theft

- **Component:** Browser
- **STRIDE category:** Spoofing
- **Severity:** High
- **Status:** Open

### Description

An attacker who steals a session cookie or access token may impersonate a legitimate user.

### Validation

Verify that session cookies use `Secure`, `HttpOnly`, and an appropriate `SameSite` attribute. Check that tokens are not stored in `localStorage` unless there is a justified design reason, sessions expire correctly, and logout invalidates the active session.

### Mitigation

Use `Secure`, `HttpOnly`, and appropriate `SameSite` cookie settings. Protect against XSS, rotate tokens where appropriate, expire sessions, and invalidate compromised sessions.

## 4. Excessive Data Returned by API

- **Component:** API responses
- **STRIDE category:** Information Disclosure
- **Severity:** High
- **Status:** Open

### Description

The API may return fields or records that the current user does not need or is not authorised to access.

### Validation

Review API responses and compare returned fields with what the React application actually uses. Check whether the API exposes unnecessary personal data, internal identifiers, roles, permissions, security-related fields, or records belonging to other users.

### Mitigation

Return only required fields, enforce server-side authorisation, and use response schemas or DTOs to prevent accidental data exposure.

## 5. Missing Audit Logging

- **Component:** Backend API
- **STRIDE category:** Repudiation
- **Severity:** Medium
- **Status:** Open

### Description

Sensitive actions may not be recorded with enough information to determine who performed them and when.

### Validation

Review whether sensitive backend operations create structured audit events containing:

- authenticated user ID
- action performed
- target resource
- timestamp in UTC
- request or correlation ID
- success or failure outcome
- source IP where reliable
- useful application context

Verify that audit logs cannot be modified by ordinary users and that related events can be traced across requests.

### Mitigation

Create structured audit logs for security-sensitive operations. Do not log passwords, full session tokens, access tokens, sensitive personal data, payment details, or complete request bodies by default. Protect logs from tampering and define retention and access-control rules.

## 6. Sensitive Information Exposed in Browser Console

- **Component:** React Frontend
- **STRIDE category:** Information Disclosure
- **Severity:** Medium
- **Status:** Open

### Description

The React application may expose sensitive information through `console.log()`, `console.info()`, `console.debug()`, or unhandled error messages. This may include API responses, tokens, user data, internal identifiers, feature flags, or implementation details.

### Validation

Use SAST and manual code review to identify sensitive console logging. Use DAST or a manual runtime review as supporting checks by inspecting the browser console and production error output.

### Mitigation

Remove sensitive debug logging from production builds. Never log tokens, credentials, personal data, or complete API responses. Use controlled logging utilities and environment-based log levels.

## 7. Injection in Database Query

- **Component:** Database queries
- **STRIDE category:** Tampering
- **Severity:** High
- **Status:** Open

### Description

User-controlled input may be included in database queries without safe parameterisation or strict query construction. This could allow an attacker to alter query logic, access unauthorised data, modify records, bypass authentication, or trigger database errors.

### Validation

Review the backend data-access code and verify that:

- untrusted input is never concatenated into query strings
- prepared statements or parameterised queries are used consistently
- ORM raw query usage is identified and reviewed
- inputs with limited valid values use strict allowlists
- database accounts follow least privilege
- SQL or database error details are not exposed to users
- NoSQL input types and schemas reject operator-shaped input

Use SAST and manual code review to identify unsafe query construction. Where authorised, test relevant endpoints with controlled injection payloads.

### Mitigation

Use parameterised queries or prepared statements. For NoSQL databases, enforce strict input types and schemas, reject operator-shaped input, and avoid passing user-controlled objects directly into query filters. Apply least-privilege permissions to the database account.

## 8. Sensitive Data Returned from Database

- **Component:** Query results
- **STRIDE category:** Information Disclosure
- **Severity:** Medium
- **Status:** Open

### Description

Database queries may return more records or fields than the current user is authorised to access.

### Validation

Review selected fields and query scope. Check whether raw database objects are returned, verify authorisation before data retrieval, and compare returned data with what the API and frontend actually need.

### Mitigation

Enforce authorisation before and after data retrieval, limit selected fields, use scoped queries, and avoid returning raw database objects directly to the client.
