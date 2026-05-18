# Lab 03 - PortSwigger: User ID Controlled by Request Parameter

> **Platform:** PortSwigger Web Security Academy  
> **Topic:** Access Control / IDOR  
> **Status:** Completed  
> **Type:** Lab notes, not a full walkthrough

---

## Practice Focus

This lab demonstrated a classic horizontal privilege escalation issue.

The application used a user-controlled request parameter to decide which user's account data to return.

---

## Vulnerability Pattern

A request contains a user identifier, for example:

```http
GET /my-account?id=wiener
```

If the backend accepts a changed identifier and returns another user's data, access control is broken.

---

## What I Tested

- Which request parameter controlled the user/account page.
- Whether changing the identifier changed the returned account data.
- Whether the backend checked that the requested account belonged to the logged-in user.
- How the response changed after modifying the parameter.

---

## Root Cause

The backend trusted a user-controlled identifier without verifying that the authenticated user was allowed to access the requested account.

---

## Impact

An attacker could access another user's account data or sensitive information.

In real applications, similar issues could expose:

- personal information,
- API keys,
- invoices,
- orders,
- private messages,
- account settings.

---

## Remediation

The backend should:

- identify the current user from the session or token,
- ignore client-controlled user IDs where possible,
- check ownership before returning data,
- return `403` or `404` for unauthorized access.

Unsafe pattern:

```ts
getAccount(req.query.id)
```

Safer pattern:

```ts
getAccountForUser(req.session.userId)
```

---

## Main Takeaway

```text
Changing a user ID should not allow access to another user's account.
```
