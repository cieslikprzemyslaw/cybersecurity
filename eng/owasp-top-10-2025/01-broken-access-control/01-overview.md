# A01 Broken Access Control

## Scope

This note maps my practical access-control learning to **OWASP Top 10 2025 A01 Broken Access Control**.

It is written from the perspective of a Frontend Engineer moving into AppSec. The focus is not only on exploiting a lab, but on understanding how this type of issue appears in real applications and how it should be reviewed, fixed and regression-tested.

## What this category means

Broken Access Control happens when an application does not correctly enforce what an authenticated or unauthenticated user is allowed to do.

A useful way to separate the concepts is:

- **Authentication** asks: who are you?
- **Authorization / Access Control** asks: are you allowed to perform this action on this resource?

In the lab I completed, authentication worked because the normal user was logged in correctly. The failure was authorization: the backend did not verify that the logged-in user had administrator privileges before processing the final role-change request.

## Why it matters

Access-control failures can allow users to act outside their intended permissions.

Depending on the feature, this can lead to:

- viewing another user's data,
- changing another user's account details,
- accessing admin-only functionality,
- modifying roles or permissions,
- downloading files outside the intended scope,
- triggering privileged backend functionality,
- reaching internal resources through server-side functionality,
- changing application state without proper authorization.

## Abuse case

A low-privileged user discovers or reconstructs an admin-only request and sends it directly to the backend.

Because the final state-changing endpoint does not enforce authorization, the user upgrades their own role to administrator and gains access to admin-only functionality.

## Common examples

Examples of Broken Access Control include:

- a normal user accessing `/admin` functionality,
- modifying another user's account by changing a request parameter,
- accessing another user's invoice, document or profile by changing an ID,
- performing admin actions by sending direct API requests,
- missing authorization on the final step of a multi-step workflow,
- relying on frontend route guards or hidden buttons as the main protection,
- relying on a previous step in a workflow instead of authorizing the endpoint that performs the action,
- allowing access to files outside the intended directory,
- allowing backend requests to reach internal-only destinations where access boundaries are not enforced.

## Common root causes

Common root causes include:

- authorization checks are missing from one or more backend endpoints,
- access control is applied only in the UI,
- access control is applied only at the start of a multi-step process,
- the backend trusts request parameters such as `username`, `userId`, `role` or `confirmed`,
- ownership checks are missing or inconsistent,
- admin functionality is protected by obscurity rather than permission checks,
- application logic assumes users can only reach endpoints through the intended UI flow,
- there is no central authorization policy or reusable guard,
- testing focuses on the happy path and does not test direct requests from lower-privileged users.

## Impact

The impact depends on the affected feature.

For the completed lab, the impact was **privilege escalation**:

> A normal authenticated user could directly send the final confirmation request and upgrade their own account to administrator.

In a real application, this could lead to:

- full access to admin-only features,
- unauthorized role changes,
- user management abuse,
- unauthorized data access,
- unauthorized data modification,
- loss of trust in account and permission boundaries,
- possible wider compromise if admin features expose sensitive operations.

## Related internal notes

This category connects with several topics I have already studied:

- [Broken Access Control / IDOR](../../key-web-vulnerabilities/02-broken-access-control-idor/README.md) - direct object access where the backend fails to check ownership or permission.
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md) - related when sensitive state-changing actions rely only on the user's session.
- [Path Traversal / File Access Bugs](../../key-web-vulnerabilities/06-path-traversal-file-access/README.md) - related when users can access files outside the intended directory or authorization boundary.
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md) - related when backend request functionality can reach internal or privileged network locations.
- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md) - useful background for separating authentication failures from authorization failures.

## My takeaway as a Frontend Engineer

Frontend controls are important for user experience, but they are not a security boundary.

A React route guard, hidden admin link, disabled button or conditional UI check can help prevent accidental use, but users still control the browser. They can send requests directly using Burp Suite, Postman, curl, DevTools or custom scripts.

Real access control must be enforced on the backend endpoint that performs the action.

A good frontend implementation should still:

- avoid exposing unnecessary admin UI,
- avoid trusting client-side role values,
- handle `401` and `403` responses clearly,
- avoid leaking sensitive functionality through predictable links,
- support the backend access-control model cleanly.

But it cannot replace backend authorization.

## What good looks like

A safer implementation should:

- enforce authorization server-side on every sensitive endpoint,
- apply deny-by-default for protected actions,
- check both role and ownership where needed,
- avoid trusting user-controlled parameters for authorization decisions,
- re-check permissions on final confirmation steps,
- keep authorization logic centralized where possible,
- test direct backend requests from lower-privileged users,
- ensure the state does not change when authorization fails,
- log suspicious unauthorized attempts for sensitive actions.

## External references

- OWASP Top 10 2025 A01 Broken Access Control: https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/
- PortSwigger Web Security Academy: Access control vulnerabilities: https://portswigger.net/web-security/access-control
- PortSwigger lab completed: Multi-step process with no access control on one step: https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step
