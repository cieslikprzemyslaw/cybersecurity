# A01 Checklist: Broken Access Control

Use this checklist during code review, API review, manual testing or AppSec review.

The main question is:

> Can a user perform an action or access a resource outside their intended permissions?

## 1. Understand the feature

Before testing, identify:

- What action is being performed?
- Is the action state-changing?
- Is the action read-only but sensitive?
- Who should be allowed to perform it?
- Is the rule based on role, ownership, organisation, tenant, status or workflow state?
- Which backend endpoint actually performs the action?
- Is the feature part of a multi-step workflow?

Examples of sensitive actions:

- role changes,
- account deletion,
- user management,
- viewing invoices or documents,
- changing email or password,
- downloading files,
- publishing content,
- approving/rejecting submissions,
- payment or billing changes,
- admin moderation actions.

## 2. Authentication vs authorization

Check that the team is not mixing up authentication and authorization.

Authentication questions:

- Is the user logged in?
- Is the session valid?
- Is the token valid?

Authorization questions:

- Is this logged-in user allowed to perform this action?
- Is this user allowed to access this specific resource?
- Is this user allowed to modify this specific object?
- Is this user allowed to complete this workflow step?

A user can be correctly authenticated and still not authorized.

## 3. Code review questions

Ask:

- Where is the authorization check performed?
- Is it enforced on the backend?
- Is it enforced on every sensitive endpoint?
- Is the final state-changing endpoint protected?
- Are ownership checks performed server-side?
- Are role checks performed server-side?
- Is the logic deny-by-default?
- Is there a central guard, middleware, policy or service-level check?
- Are checks duplicated inconsistently across controllers?
- Can a user change `userId`, `username`, `role`, `tenantId`, `accountId`, `orgId` or `confirmed` in the request?
- Does the backend trust anything that comes from the client for authorization decisions?
- Are authorization checks covered by automated tests?

## 4. Testing questions

For each sensitive request, test:

- What happens as an unauthenticated user?
- What happens as a normal authenticated user?
- What happens as the resource owner?
- What happens as a different user?
- What happens as an admin?
- What happens if IDs are changed?
- What happens if the HTTP method is changed?
- What happens if the request is sent directly without using the UI?
- What happens if only the final confirmation step is sent?
- What happens if the request is replayed?
- What happens if optional-looking parameters are modified?
- Does the application return `403 Forbidden` for authenticated but unauthorized users?
- Does the application keep the state unchanged after rejection?

## 5. Multi-step workflow checklist

For workflows such as role changes, approvals, account deletion or publishing:

- Identify every request in the workflow.
- Identify the request that actually changes state.
- Test every step with a lower-privileged user.
- Test the final confirmation step directly.
- Check whether the backend relies on `confirmed=true` or similar client-controlled values.
- Check whether earlier authorization is assumed rather than re-checked.
- Confirm that unauthorized final-step requests fail.
- Confirm that the underlying state does not change.
- Confirm that suspicious attempts are logged where appropriate.

## 6. Frontend-specific checks

Frontend checks are useful, but they are not sufficient security controls.

Review:

- Are admin links hidden from normal users?
- Are protected routes guarded for UX?
- Does the frontend handle `401` and `403` correctly?
- Does the UI accidentally expose admin endpoint paths?
- Does the UI rely on client-side role values that can be modified?
- Does the UI prevent accidental actions?

But also confirm:

- The backend enforces the same permission model.
- API endpoints reject unauthorized direct requests.
- Sensitive actions do not depend only on hidden buttons, disabled controls or route guards.

## 7. Developer red flags

Red flags include:

- "The user cannot see the button, so they cannot do it."
- "Only admins know this URL."
- "The first step checks admin, so the confirmation step is fine."
- "The frontend sends the role, so the backend trusts it."
- "The request comes from our UI, so it is safe."
- "The endpoint is not linked anywhere."
- "We check permissions on the page load, not on the action."
- "This is only an internal endpoint."
- "The user ID comes from a hidden field."
- "The role is disabled in the UI, so the user cannot change it."

## 8. Remediation checklist

A safer implementation should:

- enforce authorization server-side,
- use deny-by-default for sensitive actions,
- check the current authenticated user, not just request parameters,
- check role and ownership where appropriate,
- protect every endpoint that performs a sensitive action,
- protect final confirmation steps in multi-step workflows,
- centralize authorization logic where possible,
- avoid trusting client-controlled values for permission decisions,
- return `403 Forbidden` for authenticated but unauthorized users,
- ensure the application state does not change after unauthorized requests,
- log suspicious unauthorized attempts for high-risk actions,
- add regression tests for direct backend requests.

## 9. Safe implementation notes

Good patterns:

- shared authorization middleware or guards,
- policy-based checks such as `canUpdateUser(currentUser, targetUser)`,
- object-level authorization checks,
- tenant/organisation scoping in queries,
- server-side role checks before state changes,
- explicit allowlists of permitted actions,
- tests for both allowed and denied cases.

Avoid:

- relying only on frontend route guards,
- trusting `userId` from the request body,
- trusting `role` from the client,
- trusting `confirmed=true`,
- protecting only the first workflow step,
- assuming users follow the intended UI flow,
- using obscurity as access control.

## 10. Review outcome

A feature should pass the A01 review only if:

- unauthorized users cannot access protected resources,
- unauthorized users cannot perform protected actions,
- direct requests are rejected,
- final workflow steps are protected,
- state remains unchanged after rejected requests,
- expected users can still complete the allowed workflow,
- regression tests exist for both positive and negative access-control cases.
