# Broken Access Control & IDOR — Overview

> **Topic:** Broken Access Control, IDOR, object references, server-side authorization  
> **Status:** Reference notes, not a full walkthrough  
> **Sources:** TryHackMe + PortSwigger Web Security Academy + OWASP

---

## TL;DR

**Broken Access Control happens when the application does not correctly enforce what an authenticated user is allowed to access or do.**

**IDOR is not about guessing an ID.**  
The real issue is that the backend fails to verify whether the current user is allowed to access the object behind that ID.

```text
Authenticated user + modified object reference + missing server-side authorization = IDOR
```

---

## Why This Matters

Authentication and authorization are different controls.

**Authentication / AuthN** answers:

> Who are you?

**Authorization / AuthZ** answers:

> What are you allowed to access or do?

A user can be correctly logged in but still should not be allowed to access another user's profile, invoice, file, transcript, order, admin action, or account setting.

Broken Access Control is mainly an **authorization** problem.

---

## What is Broken Access Control?

Broken Access Control means that the application does not properly enforce access rules on the server side.

Examples:

- a normal user can access admin functionality,
- user A can view or modify user B's data,
- a user can download a file that belongs to someone else,
- changing a value in a request gives access to another object,
- the frontend hides a button but the backend still accepts the action,
- an API endpoint trusts `userId`, `role`, or `isAdmin` from the request.

The key issue is usually not whether the user is logged in.  
The key issue is whether that user is allowed to access **that exact object** or perform **that exact action**.

---

## What is IDOR?

**IDOR** stands for **Insecure Direct Object Reference**.

It happens when an application exposes a direct reference to an object and does not verify whether the current user is allowed to access it.

Example pattern:

```http
GET /account?id=123
```

Changed to:

```http
GET /account?id=124
```

If the server returns another user's account data, the backend is not checking ownership correctly.

The problem is not that the user changed the ID. Users can always modify requests.  
The problem is that the backend trusted the object reference without checking authorization.

---

## Object References Are Not Always `?id=123`

Object references can appear in many places:

- query parameters, for example `?id=123`,
- URL paths, for example `/users/123`,
- request body, for example `{ "userId": 123 }`,
- hidden form fields,
- cookies,
- headers,
- file names, for example `1.txt` or `invoice.pdf`,
- download links,
- API endpoints,
- GraphQL variables,
- UUIDs,
- encoded values, for example Base64.

One useful lesson from the labs was that IDOR is about **access to an object**, not only numeric IDs.

In one PortSwigger lab, the insecure object reference was a transcript file reference, not a classic `id` parameter.

---

## Types of Access Control Issues

### Horizontal Privilege Escalation

A user accesses data or actions belonging to another user at the same privilege level.

Example:

```text
User A reads User B's profile.
```

### Vertical Privilege Escalation

A lower-privileged user performs actions reserved for a higher-privileged user.

Example:

```text
A normal user accesses an admin-only function.
```

### Context-Dependent Access Control

The action may be allowed in one context but not another.

Example:

```text
A user may edit their own profile, but not another user's profile.
```

---

## Practice Covered

The practical work for this topic focused on:

- access control basics and AuthN vs AuthZ thinking,
- changing object references and observing backend behaviour,
- classic user-controlled IDOR through request parameters,
- IDOR through file and transcript references, not only `?id=123`.

---

## Testing Approach

A simple workflow:

1. Log in as a normal user.
2. Find requests that reference objects or actions.
3. Send the request to Burp Repeater.
4. Change the object reference or action parameter.
5. Compare the response.
6. Check whether the server returns another user's data or correctly denies access.

Useful question:

> Is the backend checking that this resource belongs to the current user?

A strong real-world testing pattern is to use two test accounts:

```text
Account A creates a resource.
Account B attempts to access or modify Account A's resource.
```

If Account B can access it, that is a strong sign of Broken Access Control.

---

## Expected Secure Behaviour

If a user is not allowed to access a resource, the server should not return the data.

| Situation | Expected Response |
|---|---|
| User is not logged in | `401 Unauthorized` or redirect to login |
| User is logged in but lacks permission | `403 Forbidden` |
| Application does not want to reveal resource existence | `404 Not Found` |

Unsafe behaviour:

```text
200 OK + another user's data
```

---

## Developer Remediation

Good backend practices:

- Do not trust `userId`, `accountId`, `fileId`, `invoiceId`, `role`, or `isAdmin` from the request.
- Get the current user from the session or token.
- Check resource ownership before returning data.
- Check roles and permissions for every sensitive action.
- Apply deny-by-default access control.
- Apply the principle of least privilege.
- Add automated tests for access control.
- Log suspicious access attempts without exposing sensitive data.

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
- defense in depth,
- verify access per object and per action.

---

## Main Takeaway

The frontend is not a security boundary.

A React app can hide buttons, disable inputs, or remove links from the UI, but a user can still send a modified request using Burp, Postman, curl, DevTools, or a script.

The backend must always answer:

> Is this authenticated user allowed to access this exact object and perform this exact action?

That is the core lesson from Broken Access Control and IDOR.
