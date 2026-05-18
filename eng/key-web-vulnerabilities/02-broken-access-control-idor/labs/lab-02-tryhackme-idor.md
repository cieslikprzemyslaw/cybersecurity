# Lab 02 Summary — TryHackMe IDOR

> **Platform:** TryHackMe  
> **Topic:** Insecure Direct Object Reference  
> **Status:** Completed  
> **Type:** Lab notes, not a full walkthrough

---

## Practice Focus

This room focused on changing object references and observing whether the backend correctly enforces authorization.

Examples of object references include:

- user IDs,
- account IDs,
- document IDs,
- file names,
- API object identifiers.

---

## Key Concept

IDOR happens when the application exposes a reference to an object and does not verify whether the current user is allowed to access that object.

Example pattern:

```http
GET /resource?id=123
```

If changing the ID returns another user's resource, the backend is missing an ownership or permission check.

---

## Key Points

- IDOR is a type of Broken Access Control.
- IDOR is not only about numeric IDs.
- Object references may appear in URLs, paths, request bodies, headers, cookies, or encoded values.
- The problem is not that the user can modify a request.
- The problem is that the backend trusts the modified reference.

---

## Developer Takeaway

The backend should not rely on the object reference alone.

It should check:

```text
requested object + current authenticated user + permission/ownership
```

---

## Remediation Notes

Safer implementation patterns:

- derive the current user from the session/token,
- check object ownership before returning data,
- avoid trusting `userId` or `accountId` from the client,
- use deny-by-default behaviour,
- test access with two accounts.

---

## Main Takeaway

```text
IDOR is about missing authorization for the object behind the ID.
```
