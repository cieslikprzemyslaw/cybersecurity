# Broken Access Control & IDOR — AppSec Learning Notes

> **Learning path:** Frontend Engineer → Application Security  
> **Sources:** TryHackMe + PortSwigger Web Security Academy  
> **Topic:** Broken Access Control, IDOR, object references, server-side authorization  
> **Status:** Learning notes, not a full lab walkthrough

---

## TL;DR

**IDOR is not about guessing an ID.**  
The real issue is that the backend fails to check whether the authenticated user is allowed to access the object behind that ID.

```text
Authenticated user + modified object reference + missing server-side authorization = IDOR
```

---

## Labs Covered

| Platform | Lab / Topic | What I Practiced |
|---|---|---|
| TryHackMe | Broken Access Control | Access control basics and AuthN vs AuthZ thinking |
| TryHackMe | IDOR | Changing object references and observing backend behaviour |
| PortSwigger | User ID controlled by request parameter | Classic user-controlled IDOR through a request parameter |
| PortSwigger | Insecure direct object references | IDOR through transcript/files, not only `?id=123` |

---

## Key Concepts

### Authentication vs Authorization

**Authentication / AuthN** asks:

> Who are you?

**Authorization / AuthZ** asks:

> What are you allowed to access or do?

IDOR and Broken Access Control are mainly **authorization** problems. A user can be correctly logged in, but still should not be allowed to access another user's data, files, invoices, chat logs, or account details.

---

## What is Broken Access Control?

Broken Access Control happens when an application does not properly enforce access rules on the server side.

Examples:

- a normal user can access admin functionality,
- user A can read user B's profile,
- a user can download another user's file,
- changing an ID in the URL returns another user's data,
- the frontend hides a button but the backend still accepts the request.

---

## What is IDOR?

**IDOR** stands for **Insecure Direct Object Reference**.

It happens when the application exposes a direct reference to an object and does not verify whether the current user is allowed to access it.

Example pattern:

```http
GET /account?id=123
```

Changed to:

```http
GET /account?id=124
```

If the server returns another user's account data, this is an IDOR issue.

---

## Object References Are Not Always `?id=123`

One of the most useful lessons was that object references can appear in many places:

- query parameters: `?id=123`,
- URL paths: `/users/123`,
- request body: `{ "userId": 123 }`,
- hidden form fields,
- cookies,
- headers,
- file names: `1.txt`, `invoice.pdf`,
- download links,
- API endpoints,
- GraphQL variables,
- encoded values, such as Base64,
- UUIDs.

In one PortSwigger lab, the insecure object reference was not a user ID. It was a transcript file reference. That was a useful reminder that IDOR is about access to an object, not only numeric IDs.

---

## How I Think About Testing for IDOR

A simple testing workflow:

1. Log in as a normal user.
2. Find requests that reference objects.
3. Send the request to Burp Repeater.
4. Change the object reference.
5. Compare the response.
6. Check whether the server returns another user's data or correctly denies access.

Useful question:

> Is the backend checking that this resource belongs to the current user?

A very strong test pattern is using two accounts:

```text
Account A creates a resource.
Account B attempts to access or modify Account A's resource.
```

If Account B can access it, that is a strong sign of Broken Access Control.

---

## Expected Secure Behaviour

If a user is not allowed to access a resource, the server should not return the data.

Possible safe responses:

| Situation | Expected response |
|---|---|
| User is not logged in | `401 Unauthorized` or redirect to login |
| User is logged in but lacks permission | `403 Forbidden` |
| Application does not want to reveal resource existence | `404 Not Found` |

Unsafe response:

```text
200 OK + another user's data
```

---

## Developer Remediation

Good backend practices:

- Do not trust `userId`, `accountId`, `fileId`, `invoiceId`, `role`, or `isAdmin` from the request.
- Get the current user from the session or token.
- Check resource ownership before returning data.
- Check permissions on every sensitive action.
- Apply the principle of least privilege.
- Add automated tests for access control.
- Log suspicious attempts to access other users' resources.

Unsafe pattern:

```ts
getInvoice(req.query.invoiceId)
```

Safer pattern:

```ts
getInvoiceForUser(req.session.userId, req.query.invoiceId)
```

The important difference is that the backend checks both:

1. which object was requested,
2. whether the current user is allowed to access it.

---

## OWASP / AppSec Mapping

Relevant category:

- **OWASP Top 10: Broken Access Control**

Related principles:

- never trust user input,
- principle of least privilege,
- server-side authorization,
- deny by default,
- defense in depth.

---

## Review Checklist

When reviewing a feature, I should ask:

- Does this request contain an object reference?
- Can the user modify that reference?
- Is the current user taken from the session/token, not from request data?
- Does the backend check ownership?
- Does the endpoint behave correctly for another user?
- Does the frontend hide something that the backend still allows?
- Would the correct response be `403`, `404`, or redirect to login?

---

## Main Takeaway

The frontend is not a security boundary.

A React app can hide buttons, disable inputs, or remove links from the UI, but a user can still send a modified request using Burp, Postman, curl, or a script.

The backend must always answer:

> Is this authenticated user allowed to access this exact object and perform this exact action?

That is the core lesson from Broken Access Control and IDOR.
