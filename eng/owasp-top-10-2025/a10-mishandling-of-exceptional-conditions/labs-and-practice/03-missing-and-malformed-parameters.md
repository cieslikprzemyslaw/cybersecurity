# Practice 03: Missing and Malformed Parameters

## Scenario

Endpoint:

```http
PATCH /api/assessments/:assessmentId/status
```

Allowed status values:

```text
draft
in-progress
completed
archived
```

The reviewed code only checked:

```ts
if (!status) {
  return res.status(400).json({ error: "status is required" });
}
```

## Test cases

| Body | Expected result | Reason |
| --- | --- | --- |
| `{}` | reject, `400` | missing required field |
| `{ "status": "" }` | reject, `400` | empty value |
| `{ "status": null }` | reject, `400` | null is not a valid status string |
| `{ "status": ["completed"] }` | reject, `400` | wrong data type |
| `{ "status": "admin" }` | reject, `400` | unknown enum value |
| `{ "status": "completed" }` | accept, `200`, only if auth/authz and state transition rules pass | valid input value |

## What I corrected

I initially treated a missing authorization check as a possible `500`. The corrected model is:

```text
missing auth/authz check -> usually A01 Broken Access Control or A06 Insecure Design
security dependency exists but fails and the app allows the action -> A10 fail-open
```

A valid request from an unauthenticated user should not be `500`.

- Not authenticated -> `401 Unauthorized`
- Authenticated but not allowed -> `403 Forbidden`, or intentionally `404 Not Found`
- Valid and allowed -> `200 OK`
- Unexpected server failure -> controlled `500`

## A10 learning point

Missing, null, empty, wrong-type or unknown-enum values should be rejected as expected validation failures. They should not flow into business logic and cause strange exceptions, unsafe state transitions or unexpected server behaviour.

## Safe expected behaviour

The backend should:

- validate the body shape,
- validate `status` is a string,
- validate `status` is one of the allowed values,
- reject unknown fields if the API contract requires strict input,
- check authentication,
- check authorization for the specific assessment,
- check that the state transition is allowed,
- return controlled `400`, `401`, `403`/`404`, or `200`, depending on the case.

## Regression tests

Useful tests:

- `{}` returns `400`,
- empty string returns `400`,
- `null` returns `400`,
- array value returns `400`,
- object value returns `400`,
- unknown enum returns `400`,
- valid enum with unauthenticated user returns `401`,
- valid enum with unauthorized user returns `403` or intentional `404`,
- valid enum with authorized user updates the status,
- malformed input never changes the assessment status.

## Frontend takeaway

Frontend form validation is useful, but the backend must enforce the API contract. Attackers and testers can bypass the UI using Burp, ZAP, Postman, DevTools or direct scripts.
