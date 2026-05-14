# HTTP, Request/Response and Auth Basics

## 1. Overview

This note covers the foundations of how web applications communicate and where security decisions should happen.

Before testing vulnerabilities such as IDOR, SQL Injection, XSS, CSRF or Broken Access Control, I need to understand how requests and responses work. Most web security testing starts with one simple question:

> What is the client sending to the server, and how does the server respond?

From an AppSec perspective, the browser is only one way of sending requests. The same request can often be sent or modified using tools like Burp Suite, Postman, curl, DevTools or a custom script.

## 2. HTTP Request

An HTTP request is a message sent from a client to a server.

The client can be:

- a browser,
- a mobile app,
- Postman,
- Burp Suite,
- curl,
- another backend service,
- a custom script.

Example request:

```http
GET /products?id=123 HTTP/1.1
Host: example.com
Cookie: session=abc123
User-Agent: Chrome
```

A request is created when a user:

- opens a URL,
- clicks a link,
- submits a form,
- sends data to an API,
- uploads a file,
- logs in,
- changes account details,
- performs an action inside the application.

In AppSec, I should not only look at what the UI shows. I should look at what request is actually being sent.

## 3. Common HTTP Methods

HTTP methods describe the intention of the request.

| Method | Common purpose |
|---|---|
| `GET` | Retrieve data |
| `POST` | Send or create data |
| `PUT` | Replace or update a resource |
| `PATCH` | Partially update a resource |
| `DELETE` | Delete a resource |
| `OPTIONS` | Ask the server which methods are allowed |
| `HEAD` | Similar to GET, but without the response body |

Examples:

```http
GET /api/users/123
POST /api/users
PUT /api/users/123
PATCH /api/users/123
DELETE /api/users/123
```

From a security perspective, every method matters. A hidden or unprotected `PUT`, `PATCH` or `DELETE` endpoint can be more dangerous than a simple `GET` request because it may change or remove data.

## 4. HTTP Response

An HTTP response is the server's answer to a request.

A response usually contains:

- status code,
- headers,
- response body,
- cookies set by the server,
- error messages,
- redirects,
- JSON, HTML, files or other data.

Example response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: session=abc123; HttpOnly; Secure
```

```json
{
  "id": 123,
  "name": "Product"
}
```

Common status codes:

| Status code | Meaning |
|---|---|
| `200 OK` | Request succeeded |
| `201 Created` | Resource was created |
| `302 Found` | Redirect |
| `400 Bad Request` | Invalid request |
| `401 Unauthorized` | User is not authenticated |
| `403 Forbidden` | User is authenticated but not allowed |
| `404 Not Found` | Resource not found |
| `405 Method Not Allowed` | HTTP method is not allowed |
| `500 Internal Server Error` | Server-side error |

When testing security, the status code is important, but it is not enough. I should also check the response body because sensitive data can sometimes be returned even when the UI does not display it.

## 5. What the User Can Control

From an AppSec perspective, I should assume that the user can modify everything sent in the request.

User-controlled parts may include:

- HTTP method: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`,
- URL path,
- query parameters,
- request body,
- headers,
- cookies,
- tokens,
- form values,
- hidden form fields,
- resource IDs,
- uploaded files,
- content type.

Example:

```http
GET /profile?id=123
```

A user may try to change it to:

```http
GET /profile?id=124
```

Or change the method:

```http
DELETE /profile/123
```

The important rule is:

> Never trust user input.

Even if something comes from a normal browser, it can still be modified before it reaches the server.

## 6. What the Server Must Control

The server should be responsible for security decisions.

The server must control:

- authentication,
- authorization,
- session validation,
- access to resources,
- server-side validation,
- what data is returned,
- what actions are allowed,
- error handling,
- cookie creation,
- database queries,
- role and permission checks.

For example, if the request says:

```json
{
  "role": "admin"
}
```

the server should not trust that value. The server should check the user's real role from a trusted source, such as the session or database.

## 7. GET vs POST Security Misconception

A common misconception is that `POST` is more secure than `GET`.

That is not true.

`GET` usually retrieves data:

```http
GET /product?id=1
```

`POST` usually sends data:

```http
POST /login
```

But security does not come from the method itself.

Both `GET` and `POST` can be vulnerable to:

- SQL Injection,
- XSS,
- IDOR,
- Broken Access Control,
- CSRF,
- Command Injection,
- sensitive data exposure.

Also, methods like `PUT`, `PATCH` and `DELETE` can introduce serious risks if access control is missing.

Example:

```http
DELETE /api/users/124
```

If a normal user can delete another user's account, the problem is not the HTTP method. The problem is missing authorization on the server side.

## 8. Authentication vs Authorization

Authentication and authorization are different concepts.

### Authentication

Authentication means verifying identity.

The question is:

> Who are you?

Examples of authentication:

- username and password,
- MFA,
- session cookie,
- JWT,
- API token,
- magic link.

### Authorization

Authorization means checking permissions.

The question is:

> What are you allowed to do?

Example:

```http
GET /profile/123
```

Changed to:

```http
GET /profile/124
```

If user A can see user B's profile by changing the ID, the user is authenticated but not properly authorized.

This is a Broken Access Control issue and may also be an IDOR vulnerability.

## 9. Cookies and Sessions

A session cookie often stores a session identifier.

Example:

```http
Cookie: sessionId=abc123
```

The server must check:

- whether the session exists,
- whether the session is valid,
- whether the session has expired,
- which user the session belongs to,
- what permissions the user has.

The server should not blindly trust cookies because users can try to:

- modify them,
- copy them,
- delete them,
- replace them,
- reuse old values,
- send fake values.

Example:

```http
Cookie: sessionId=abc123
```

Changed to:

```http
Cookie: sessionId=test
```

A secure application should reject invalid or expired session values.

## 10. Key Takeaways

- HTTP requests are messages sent from the client to the server.
- HTTP responses are the server's answer.
- Users can modify request methods, paths, parameters, body, headers, cookies and tokens.
- `GET`, `POST`, `PUT`, `PATCH` and `DELETE` can all be security-relevant.
- `POST` is not automatically safer than `GET`.
- Authentication means verifying identity.
- Authorization means checking permissions.
- A logged-in user is not automatically allowed to access every resource.
- Security decisions must be enforced on the server side.
- Never trust user-controlled input.

## 11. Practical AppSec Checklist

When looking at a request, I should ask:

- What HTTP method is being used?
- Can the method be changed?
- What path is being requested?
- Are there any IDs in the URL, query string or body?
- Is the user allowed to access this resource?
- Is the user allowed to perform this action?
- Is the server validating the input?
- Does the response expose more data than needed?
- Are cookies or tokens used correctly?
- What happens if I change the ID, method, cookie or body?

The main mindset:

> Do not only ask whether the request works. Ask whether the user should be allowed to make it.
