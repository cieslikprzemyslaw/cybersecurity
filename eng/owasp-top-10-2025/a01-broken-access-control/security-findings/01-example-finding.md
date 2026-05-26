# Example Security Finding: Missing Authorization on Final Role-Upgrade Confirmation Step

## Summary

A normal authenticated user can upgrade their own account to administrator by directly sending the final confirmation request in a multi-step role-management workflow.

The first step is protected, but the final state-changing confirmation step does not enforce the same authorization requirement.

## Affected area

- Feature: Admin role management
- Endpoint: `POST /admin-roles`
- Affected action: User role upgrade
- User type: Normal authenticated user
- OWASP mapping: A01 Broken Access Control

## Severity

**High**

A low-privileged user can escalate to administrator. Depending on the admin feature set, this may expose user management, sensitive data or other privileged operations.

## Risk / Impact

An attacker with a normal account can bypass the protected first step of the role-upgrade workflow and directly submit the final confirmation request.

Potential impact:

- privilege escalation from normal user to administrator,
- access to admin-only functionality,
- unauthorized user management,
- unauthorized changes to roles or permissions,
- possible access to sensitive data or administrative actions,
- loss of integrity of the permission model.

## Root cause

The backend authorizes the initial role-upgrade step, but not the final confirmation step. The final endpoint appears to treat `confirmed=true` as proof that the user already passed the intended admin workflow.

That assumption is unsafe because users can send backend requests directly without using the UI.

## Evidence

### Request rejected on the first step

A normal authenticated user sends the first role-upgrade request:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

username=wiener&action=upgrade
```

The server rejects the request:

```http
HTTP/2 401 Unauthorized
```

This shows that the first step has an access-control check.

### Final confirmation request accepted

The same normal authenticated user sends the final confirmation request directly:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
```

The server accepts the request:

```http
HTTP/2 302 Found
Location: /admin
```

The user is redirected to the admin area, indicating successful privilege escalation.

## Expected secure behaviour

A normal authenticated user must not be able to perform administrator-only role changes.

The final confirmation request should return:

```http
HTTP/2 403 Forbidden
```

The application should also ensure:

- the user's role remains unchanged,
- the user cannot access `/admin`,
- the unauthorized attempt is logged if the action is considered sensitive,
- the same authorization policy applies to every step of the workflow.

## Remediation

Implement server-side authorization checks on every role-management endpoint, especially the endpoint that performs the final state-changing action.

Recommended fixes:

- Check the current authenticated user's role before processing `POST /admin-roles`.
- Do not rely on `confirmed=true` as proof that the user passed a valid admin workflow.
- Do not trust `username`, `userId`, `role` or similar client-controlled parameters for authorization decisions.
- Apply deny-by-default for role-management actions.
- Use a shared authorization middleware, guard, policy or service-level check.
- Ensure the final confirmation step performs its own authorization check.
- Return `403 Forbidden` when an authenticated user lacks the required permission.
- Add regression tests for direct requests from normal users.

## Regression test ideas

Add regression coverage for:

- a normal user sending the final confirmation request directly,
- a normal user trying to upgrade another user,
- an unauthenticated user sending role-management requests,
- an administrator completing the valid role-change workflow,
- state verification after every denied request.

Detailed test cases are captured in [04-regression-tests.md](../04-regression-tests.md).

## OWASP mapping

- OWASP Top 10 2025 A01 Broken Access Control
- Related weakness pattern: vertical privilege escalation
- Related review area: server-side authorization on state-changing endpoints

## Developer takeaway

Every backend endpoint that performs a sensitive action must enforce authorization by itself. Protecting the admin page, the first workflow step, the frontend route or the visible button is not enough.

Users can send requests directly, so the backend must verify permission at the point where the action is performed.
