# Broken Access Control & IDOR — Practical Cheat Sheet

> **Purpose:** Quick review checklist for legal labs, authorised testing, secure code review, and developer remediation.  
> **Status:** Defensive / educational. Not a full walkthrough.

---

## Core Idea

```text
Authenticated does not mean authorized.
```

A user being logged in only proves identity. It does not prove they can access a specific object or perform a specific action.

---

## Common Risky Patterns

Look for request values such as:

```text
id
userId
accountId
customerId
invoiceId
orderId
fileId
documentId
profileId
role
isAdmin
permission
owner
filename
download
transcript
```

Object references can appear in:

- query parameters,
- URL paths,
- JSON bodies,
- form fields,
- hidden inputs,
- cookies,
- headers,
- file names,
- API endpoints,
- GraphQL variables,
- encoded values.

---

## Testing Workflow

### 1. Map the feature

Ask:

- What resource is being accessed?
- What action is being performed?
- Who should be allowed to do this?
- Is this action read-only or state-changing?

### 2. Identify object references

Look for IDs, filenames, UUIDs, account references, document references, or user-related values.

Example:

```http
GET /api/invoices/123
GET /account?id=123
GET /download?file=1.txt
POST /api/profile/update
```

### 3. Test with another object

Change the object reference.

Example:

```text
id=123 -> id=124
/users/123 -> /users/124
1.txt -> 2.txt
invoiceA.pdf -> invoiceB.pdf
```

### 4. Use two accounts if possible

Best pattern:

```text
Account A creates or owns a resource.
Account B attempts to access or modify Account A's resource.
```

### 5. Compare responses

Expected safe responses:

- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- redirect to login, if not authenticated

Suspicious response:

```text
200 OK + data or action belonging to another user
```

---

## Questions to Ask in Burp Repeater

- Can I change an ID and still get data?
- Can I change a filename and download another file?
- Can I change `userId` in the request body?
- Can I change `role` or `isAdmin`?
- Can I replay an admin request as a normal user?
- Does the frontend hide an action that the backend still accepts?
- Is authorization checked for every method and route?
- Does the response leak data even after redirect?
- Does the endpoint return different behaviour for different users?

---

## Red Flags in Frontend / API Code

### Trusting IDs from the client

```ts
getProfile(req.query.userId)
```

Better:

```ts
getProfileForCurrentUser(req.session.userId)
```

### Trusting roles from the request

```ts
if (req.body.isAdmin) {
  performAdminAction()
}
```

Better:

```ts
if (currentUser.role === "admin") {
  performAdminAction()
}
```

### Frontend-only protection

```tsx
{user.role === "admin" && <DeleteUserButton />}
```

This is fine for UX, but it is not a security control. The backend must still enforce authorization.

---

## Developer Remediation Checklist

Backend should:

- derive the current user from the session or token,
- never trust `userId`, `role`, or `isAdmin` from the client,
- check object ownership before returning or modifying data,
- check permissions for every sensitive action,
- apply deny-by-default rules,
- use centralized authorization helpers or middleware where possible,
- write tests for cross-user access,
- log suspicious access attempts,
- avoid leaking sensitive data in redirects or error messages.

---

## Regression Test Ideas

For every sensitive endpoint, test:

```text
User A owns Resource A.
User B attempts to read Resource A.
User B attempts to modify Resource A.
User B should receive 403 or 404.
```

For admin-only actions:

```text
Admin can perform the action.
Normal user cannot perform the action.
Anonymous user cannot perform the action.
```

For file/object downloads:

```text
User A downloads own file.
User B cannot download User A's file.
Changing file ID/name does not expose another user's file.
```

---

## Safe Response Guide

| Scenario | Safer Response |
|---|---|
| Not logged in | `401` or login redirect |
| Logged in but not allowed | `403` |
| Avoid revealing object existence | `404` |
| Successful authorized access | `200` with only allowed data |

Avoid:

```text
200 OK + another user's data
302 redirect + sensitive data in response body
verbose errors exposing object details
```

---

## OWASP / ASVS Mapping

- OWASP Top 10: Broken Access Control
- Related principles:
  - deny by default,
  - least privilege,
  - server-side authorization,
  - object-level authorization,
  - function-level authorization,
  - secure defaults.

---

## Main Developer Takeaway

```text
The client can request anything. The server must decide what is allowed.
```

Frontend controls improve user experience, but backend authorization protects the system.

## Related Notes

- [Overview](overview.md)
- [Module summary](summary.md)
- [Labs](labs/README.md)
