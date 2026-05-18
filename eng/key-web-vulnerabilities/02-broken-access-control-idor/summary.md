# Broken Access Control & IDOR — Module Summary

> **Topic:** Broken Access Control, IDOR, object-level authorization  
> **Status:** Completed module notes  
> **Labs:** TryHackMe + PortSwigger Web Security Academy

---

## Key Outcomes

Broken Access Control is mainly an authorization failure.

Authentication confirms who the user is. Authorization decides what that user can access or do.

The core lesson from this module is:

```text
Authenticated does not mean authorized.
```

IDOR is a practical example of this problem. It happens when an application exposes an object reference and the backend does not verify whether the current user is allowed to access the object behind that reference.

---

## Key Concepts

- IDOR is a type of Broken Access Control.
- Object references are not always simple numeric IDs.
- Object references can appear in query parameters, URL paths, request bodies, cookies, headers, filenames, download links, GraphQL variables, or encoded values.
- Frontend-only restrictions are useful for UX but are not security controls.
- The backend must check authorization for every sensitive object and action.
- Safe access control should deny by default.

---

## Labs Completed

| Lab | Platform | Focus |
|---|---|---|
| [Lab 01](labs/lab-01-tryhackme-broken-access-control.md) | TryHackMe | AuthN vs AuthZ, access control categories, server-side enforcement |
| [Lab 02](labs/lab-02-tryhackme-idor.md) | TryHackMe | Object references, IDOR basics, backend ownership checks |
| [Lab 03](labs/lab-03-portswigger-user-id-controlled-by-request-parameter.md) | PortSwigger | Horizontal privilege escalation through a user-controlled ID |
| [Lab 04](labs/lab-04-portswigger-insecure-direct-object-references.md) | PortSwigger | IDOR through transcript files and predictable file references |

---

## Practical Testing Pattern

A useful testing workflow:

1. Log in as a normal user.
2. Find requests that reference users, accounts, files, orders, invoices, or other objects.
3. Change the object reference.
4. Compare the response.
5. Confirm whether the backend checks ownership or permission.

The strongest real-world pattern is testing with two accounts:

```text
Account A owns a resource.
Account B attempts to access or modify Account A's resource.
```

If Account B can access it, the application likely has a Broken Access Control issue.

---

## Developer Takeaway

Do not trust client-controlled values such as:

```text
userId
accountId
invoiceId
fileId
role
isAdmin
filename
```

The backend should derive the current user from the session or token, then check whether that user is allowed to access the requested object or perform the requested action.

Unsafe pattern:

```ts
getInvoice(req.query.invoiceId)
```

Safer pattern:

```ts
getInvoiceForUser(req.session.userId, req.query.invoiceId)
```

---

## Final Takeaway

The client can request anything.

The server must decide what is allowed.
