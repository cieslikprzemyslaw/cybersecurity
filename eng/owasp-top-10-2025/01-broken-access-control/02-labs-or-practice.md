# A01 Labs and Practice: Broken Access Control

## Completed practice

### PortSwigger Web Security Academy

**Lab:** Multi-step process with no access control on one step  
**Category:** Access control vulnerabilities and privilege escalation  
**Status:** Completed  
**Main pattern:** Missing authorization on the final step of a multi-step admin workflow

External link:

- https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step

## Lab context

This was an intentionally vulnerable PortSwigger lab used for legal learning and documentation.

The lab contained an admin panel with a multi-step process for changing a user's role. The first step of the role-upgrade workflow was protected, but the final confirmation request was not protected correctly.

The goal was to understand the admin workflow, then test whether a normal authenticated user could directly send the final confirmation request.

## What I practised

In this lab, I practised:

- observing admin-only workflows in Burp Suite,
- separating the visible UI flow from the actual backend requests,
- identifying which request performs the real state-changing action,
- testing requests with a normal user's session,
- comparing first-step authorization with final-step authorization,
- understanding why each sensitive backend endpoint must perform its own access-control check,
- using a lab result as evidence for an AppSec-style finding.

## Important distinction

The issue was not that the user could log in as administrator.

The issue was that a normal authenticated user could directly send the final confirmation request for a privileged action.

This makes the vulnerability an authorization/access-control failure, not an authentication failure.

## Observed workflow

The admin role upgrade process had two important requests.

### Step 1: Initial role upgrade request

A normal user tried to send the first request:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

username=wiener&action=upgrade
```

The server rejected this request:

```http
HTTP/2 401 Unauthorized
```

This showed that the application had some access control on the first step of the workflow.

### Step 2: Final confirmation request

The normal user then sent the final confirmation request directly:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
```

The server accepted the request and redirected the user to the admin area:

```http
HTTP/2 302 Found
Location: /admin
```

This indicated that the final confirmation step performed the privileged action without checking that the current user had administrator privileges.

## What was vulnerable

The vulnerable request was the final confirmation request sent with a normal user's session:

```http
POST /admin-roles
Cookie: session=<normal-user-session>
Content-Type: application/x-www-form-urlencoded

action=upgrade&confirmed=true&username=wiener
```

The endpoint appeared to trust the workflow state or the `confirmed=true` parameter instead of authorizing the user at the point where the role change was performed.

## Why this is Broken Access Control

This is Broken Access Control because the application failed to verify whether the authenticated user was allowed to perform the requested action.

The user was authenticated as `wiener`, but `wiener` should not have been authorized to change roles.

The first step was protected, but the final state-changing step was not. This allowed a normal user to bypass the protected step and directly execute the privileged action.

## What confused me or was worth noticing

### Session cookie handling

The test only worked after using the current session cookie for the normal user.

This was a useful reminder that Repeater can easily contain stale cookies from another login flow. When testing access control, the session must match the user role being tested.

### First request vs final request

The first request returned `401 Unauthorized`, which initially made it look like the flow was protected.

The important lesson was that access control must be tested on every request, especially the request that actually changes state.

### `401` vs `403`

In the lab, the response for the protected request was `401 Unauthorized`.

In a real application, if the user is authenticated but lacks permission, `403 Forbidden` is usually the clearer response. `401` usually means authentication is missing or invalid, while `403` means the user is authenticated but not allowed.

### UI vs backend authorization

The normal user did not need the admin UI. The request could be sent directly using Burp Suite.

This reinforces that hiding admin UI is not enough. Backend authorization must be enforced independently.

## How I would test this in a real app

I would look for this pattern in:

- role-management workflows,
- admin user-management actions,
- approval and moderation flows,
- publishing workflows,
- account deletion flows,
- email or password change confirmation steps,
- payment or billing changes,
- any multi-step workflow where the final step changes state.

For each workflow, I would test:

- whether a lower-privileged user can call each step directly,
- whether the final confirmation endpoint checks permissions,
- whether changing parameters such as `username`, `userId`, `role`, `action` or `confirmed` changes the result,
- whether the backend returns `403 Forbidden` for authenticated but unauthorized users,
- whether the underlying state remains unchanged after an unauthorized request.

## Review result

The practical lesson from this lab is simple: the application checked the user at the start of the admin workflow, but forgot to check them again when the real change happened.

From a non-technical point of view, this is like checking someone's ID at the entrance, then letting them use a staff-only control panel later without checking whether they are actually staff.

The main takeaways:

- the user was logged in, but they were not allowed to perform an admin action,
- the final confirmation request was the important request because it changed the account role,
- the backend should check permission at the exact point where the change is made,
- hiding the admin screen in the UI would not fix this, because the request can still be sent directly,
- a good fix needs tests that confirm both the error response and that the user's role did not change.

## Related internal notes

- [Broken Access Control / IDOR](../../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)

## Future practice ideas

Possible next A01-related practice:

- PortSwigger: Method-based access control can be circumvented
- PortSwigger: URL-based access control can be circumvented
- Practical review task: inspect a sample API route map and identify which endpoints need role and ownership checks
- Practical review task: write regression test ideas for admin-only workflows
