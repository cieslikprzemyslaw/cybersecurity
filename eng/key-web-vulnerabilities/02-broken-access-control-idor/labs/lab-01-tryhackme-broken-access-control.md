# Lab 01 - TryHackMe Broken Access Control

> **Platform:** TryHackMe  
> **Topic:** Broken Access Control  
> **Status:** Completed  
> **Type:** Lab notes, not a full walkthrough

---

## Practice Focus

This room helped me understand Broken Access Control as an authorization problem.

The main focus was the difference between:

- authentication: who the user is,
- authorization: what the user is allowed to access or do.

---

## Key Concept

Broken Access Control happens when the application does not properly enforce access rules.

A user may be correctly logged in but still should not be able to:

- access another user's data,
- access admin functionality,
- modify resources they do not own,
- perform actions outside their role.

---

## Key Points

- Access control must be enforced on the server side.
- Frontend restrictions are useful for UX but are not security controls.
- Access control issues can affect both data access and actions.
- Authorization checks should happen for every sensitive endpoint.
- A safe design should deny access by default.

---

## Developer Takeaway

When implementing a feature, ask:

```text
Who is allowed to access this resource?
Who is allowed to perform this action?
Where is this enforced on the backend?
```

---

## Remediation Notes

Good practices include:

- server-side authorization checks,
- least privilege,
- deny-by-default access control,
- centralized permission checks where possible,
- automated tests for unauthorized access,
- avoiding trust in client-controlled values such as `role`, `isAdmin`, or `userId`.

---

## Main Takeaway

```text
A user being logged in does not mean they are allowed to access every object or action.
```
