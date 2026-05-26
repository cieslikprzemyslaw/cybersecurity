# A01 Regression Tests: Broken Access Control

## Purpose

Regression tests should prove that the access-control fix works and that the vulnerability cannot be reintroduced later.

For Broken Access Control, it is not enough to check that a request returns an error. Tests should also verify that the underlying application state does not change.

## Target scenario

The completed lab showed a multi-step admin role-upgrade workflow where:

- the first step rejected a normal user,
- the final confirmation step accepted a normal user's direct request,
- the normal user was upgraded to administrator.

The regression tests below are designed to prevent this pattern.

## Positive tests

### 1. Administrator can complete a valid role change

```gherkin
Given I am authenticated as an administrator
And a normal user exists
When I complete the role-upgrade workflow
Then the response should indicate success
And the target user's role should be updated
And the administrator should remain authenticated
```

### 2. Authorized workflow still works after the fix

```gherkin
Given I am authenticated as an administrator
When I use the normal UI flow to upgrade an allowed user
Then the workflow should complete successfully
And the final state should match the expected role change
```

## Negative tests

### 3. Normal user cannot start the role-upgrade process

```gherkin
Given I am authenticated as a normal user
When I send POST /admin-roles with username=<my-username>&action=upgrade
Then the response should be 403 Forbidden
And my role should remain unchanged
```

Note: Some applications may return `401` for this case, but `403` is usually clearer when the user is authenticated but not authorized.

### 4. Normal user cannot send final confirmation directly

```gherkin
Given I am authenticated as a normal user
When I send POST /admin-roles with action=upgrade&confirmed=true&username=<my-username>
Then the response should be 403 Forbidden
And my role should remain unchanged
And I should not be able to access /admin
```

### 5. Normal user cannot upgrade another user

```gherkin
Given I am authenticated as a normal user
And another normal user exists
When I send POST /admin-roles with action=upgrade&confirmed=true&username=<other-username>
Then the response should be 403 Forbidden
And the other user's role should remain unchanged
```

### 6. Unauthenticated user cannot perform role-management actions

```gherkin
Given I am not authenticated
When I send POST /admin-roles with action=upgrade&confirmed=true&username=<any-username>
Then the response should be 401 Unauthorized or redirect to login
And no role should change
```

## State verification

For every failed authorization test, verify:

- the response status is correct,
- the user's role did not change,
- no admin access was granted,
- no partial state change occurred,
- the user cannot access `/admin`,
- no sensitive data was returned.

This is important because a response can look like failure while the backend still changes state.

## Method and request tampering tests

Test whether access control is enforced consistently when the request changes.

Examples:

- change the HTTP method from `POST` to `GET`,
- remove optional-looking parameters,
- change parameter order,
- change `username`,
- change `userId` if used,
- change `role`,
- replay an old request,
- send only the final confirmation request,
- remove the `Referer` header,
- remove client-side-only fields.

The expected result should remain the same:

- unauthorized users are rejected,
- state does not change,
- admin access is not granted.

## Automation ideas

Automated tests can be written at different levels.

### API/integration test

Use an authenticated normal-user session/token and send the final confirmation request directly.

Expected result:

- `403 Forbidden`,
- role unchanged,
- admin endpoint inaccessible.

### Service/policy test

Test the authorization function directly.

Example logic:

```text
canChangeUserRole(currentUser, targetUser) should return false when currentUser is not an admin.
```

### End-to-end test

Use the browser flow to confirm that:

- normal users do not see admin role-management UI,
- direct backend requests still fail,
- admins can complete the workflow.

The direct backend request is the important part. UI-only tests are not enough.

## Manual verification

Manual verification steps:

1. Log in as a normal user.
2. Capture or reconstruct the final role-upgrade confirmation request.
3. Send the request directly using Burp Suite or Postman.
4. Confirm the response is `403 Forbidden`.
5. Refresh the session or query account details.
6. Confirm the role is still normal user.
7. Try to access `/admin`.
8. Confirm access is denied.

## Logging and alerting ideas

For sensitive role-management actions, consider logging:

- unauthorized role-change attempts,
- user identity,
- target username/user ID,
- endpoint,
- timestamp,
- source IP or request metadata,
- action attempted.

Avoid logging secrets, session cookies or sensitive tokens.

A future A09 review should check whether this type of failed access-control attempt is visible to defenders.

## Acceptance criteria

The fix is acceptable only when:

- every role-management endpoint enforces server-side authorization,
- final confirmation steps are protected,
- normal users receive `403 Forbidden` for admin-only actions,
- unauthenticated users receive `401 Unauthorized` or login redirect,
- failed requests do not change state,
- valid administrator workflows still work,
- normal user functionality is unaffected,
- tests cover both allowed and denied scenarios.
