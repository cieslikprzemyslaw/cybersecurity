# Burp Suite, Proxy and Repeater Basics

## 1. Overview

This note covers the basic Burp Suite workflow for web application security testing.

Burp Suite helps me inspect and modify communication between the browser and the server. This is important because many vulnerabilities are not visible directly in the UI. They become visible when I look at the actual HTTP requests and responses.

The basic flow is:

```text
Browser -> Burp Proxy -> Server
Server -> Burp Proxy -> Browser
```

Burp allows me to slow down, inspect, modify and replay requests manually.

## 2. What is Burp Suite?

Burp Suite is a web security testing tool.

It can be used to:

- capture HTTP requests,
- inspect HTTP responses,
- modify requests before they reach the server,
- resend requests,
- test parameters,
- test cookies and tokens,
- compare server responses,
- understand how the application communicates with the backend.

For AppSec learning, Burp is useful because it shows what is really happening behind the UI.

## 3. Proxy Concept

A proxy is an intermediary between the client and the server.

When Burp is configured as a proxy:

1. The browser sends a request.
2. The request goes to Burp first.
3. Burp can show, pause or modify the request.
4. The request is forwarded to the server.
5. The server response returns through Burp.
6. The browser receives the response.

Flow:

```text
Browser -> Burp -> Server
Browser <- Burp <- Server
```

This allows me to inspect application behaviour at the HTTP level.

## 4. Intercept

Intercept allows Burp to stop a request before it reaches the server.

When `Intercept is on`, Burp pauses the request.

At this point I can:

- read the request,
- modify the request,
- forward it,
- drop it.

| Action | Meaning |
|---|---|
| `Forward` | Sends the request to the server |
| `Drop` | Cancels the request |
| `Intercept is on` | Burp will pause matching requests |
| `Intercept is off` | Requests pass through normally |

If I do not click `Forward`, the browser may keep loading because the request is waiting inside Burp.

Intercept is useful when I want to catch a specific action, for example:

- login,
- profile update,
- password reset,
- checkout,
- file upload,
- delete action,
- admin action.

## 5. Repeater

Repeater is used to manually resend and modify requests.

This is one of the most useful Burp tools for learning AppSec because I can send the same request many times and compare how the server responds.

Example:

```http
GET /product?id=1 HTTP/1.1
Host: example.com
```

Changed in Repeater:

```http
GET /product?id=2 HTTP/1.1
Host: example.com
```

This helps test whether the server checks access to resources correctly.

Repeater is useful for testing:

- IDOR,
- Broken Access Control,
- authentication behaviour,
- authorization checks,
- input validation,
- SQL Injection,
- XSS payloads,
- parameter tampering,
- cookie/session behaviour.

## 6. What Can Be Modified in a Request?

In Burp Repeater, I can modify many parts of the request.

Examples:

- HTTP method,
- URL path,
- query parameters,
- request body,
- headers,
- cookies,
- tokens,
- content type,
- resource IDs,
- user IDs,
- account IDs,
- role-related values,
- boolean values such as `isAdmin`.

Example request:

```http
GET /profile?id=1 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Possible changes:

```http
GET /profile?id=2 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Or:

```http
POST /profile?id=1 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Or:

```http
DELETE /profile/1 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

The goal is not just to change things randomly. The goal is to understand whether the server correctly validates the request and checks permissions.

## 7. Testing Server Behaviour

When I modify a request, the server may respond in different ways.

| Response | Possible meaning |
|---|---|
| `200 OK` | Request succeeded |
| `302 Found` | Redirect, often to login or another page |
| `400 Bad Request` | Invalid request format or invalid input |
| `401 Unauthorized` | User is not logged in or session is invalid |
| `403 Forbidden` | User is logged in but not allowed |
| `404 Not Found` | Resource does not exist or is hidden |
| `405 Method Not Allowed` | HTTP method is not accepted |
| `500 Internal Server Error` | Server-side error |

A secure server does not have to accept a modified request.

For example, if I change:

```http
GET /api/users/123
```

to:

```http
DELETE /api/users/123
```

a secure server should reject it unless the user is allowed to delete that resource.

## 8. Cookies and Session Testing

Cookies are often important during web security testing because they may contain session identifiers.

Example:

```http
Cookie: session=abc123
```

In Burp, I can test what happens if I change it:

```http
Cookie: session=test
```

Possible server behaviour:

- redirects to login,
- returns `401 Unauthorized`,
- returns `403 Forbidden`,
- creates a new anonymous session,
- ignores the invalid cookie,
- returns an error,
- accepts the cookie only if it is valid and active.

The important question is:

> Does the server properly verify the session, or does it blindly trust the cookie?

## 9. Why Burp Matters in AppSec

Burp helps me think beyond the visible page.

A normal user interacts with buttons, forms and links. An AppSec tester looks at the request behind those actions.

For example, a button may trigger this request:

```http
POST /api/account/update-email
```

The important AppSec questions are:

- Can I change the email to another user's email?
- Can I change the user ID?
- Can I change the role?
- Can I remove required fields?
- Can I change the method?
- Can I reuse the request without a valid session?
- Can I perform the action as a different user?

Burp makes those questions practical.

## 10. Request Review Checklist

When reviewing a request in Burp, I should check:

- What endpoint is being called?
- What HTTP method is used?
- Are there query parameters?
- Is there a request body?
- Are there IDs such as `userId`, `accountId`, `orderId` or `profileId`?
- Are there role-related values such as `role`, `admin`, `isAdmin`?
- Are cookies or tokens present?
- What content type is used?
- What response status is returned?
- Does the response body contain sensitive data?
- Does the server check whether this user should be allowed to perform the action?

Useful mindset:

> If a value identifies a user, account, object or permission, it is worth testing carefully.

## 11. Key Takeaways

- Burp acts as a proxy between the browser and the server.
- Intercept stops requests before they reach the server.
- Forward sends the request onward.
- Drop cancels the request.
- Repeater allows manual testing of the same request multiple times.
- Request parameters, cookies, headers, body and methods can be modified.
- The server should reject requests that are invalid or unauthorized.
- Status codes and response bodies both matter.
- Burp helps reveal how the application behaves behind the UI.
- Manual request testing is a core AppSec skill.
