# Practice 01: API Error-Handling Review

## Scenario

Endpoint:

```http
POST /api/assessments/:assessmentId/evidence
```

The endpoint validates request data, checks whether the assessment exists, creates an evidence record and uses a global error handler for unexpected exceptions.

## What I reviewed

I separated expected request problems from unexpected server-side failures.

| Case | Expected response | Reason |
| --- | --- | --- |
| Missing required body fields, such as `title` or `content` | `400 Bad Request` | The request is malformed or incomplete. |
| Unsupported evidence type, such as `"admin"` | `400 Bad Request` | The enum value is not allowed by the API contract. |
| Valid-looking `assessmentId`, but no matching record exists | `404 Not Found` | The requested resource cannot be found or should not be revealed. |
| User is not authenticated | `401 Unauthorized` | The client has not proven identity. |
| User is authenticated but cannot access this assessment | `403 Forbidden` or intentionally `404 Not Found` | The user is not allowed to access or modify the resource. |
| Repository, database or dependency throws unexpectedly | controlled `500 Internal Server Error` | This is an unexpected server-side/application condition. |

## What I initially mixed up

I initially connected missing `assessmentId` with authentication. The corrected model is:

```text
wrong route / missing route segment -> often 404 route not found
malformed parameter value -> 400 Bad Request
not logged in -> 401 Unauthorized
logged in but not allowed -> 403 Forbidden, or sometimes 404 to hide the resource
valid-looking ID but no DB record -> 404 Not Found
unexpected server/dependency failure -> controlled 500
```

## A10 learning point

A generic `500` response protects the client from internal details, but it does not prove that the backend safely recovered.

The response only proves what the client received. It does not prove:

- database rollback,
- file cleanup,
- resource release,
- consistent state,
- or safe handling of side effects.

## Safe expected behaviour

The API should:

- reject invalid input before repository/database access,
- return controlled `4xx` responses for expected validation and access failures,
- route unexpected exceptions to a global fallback error handler,
- return a generic client response with a correlation/request ID,
- log enough internal context for investigation,
- avoid exposing stack traces, SQL, filesystem paths, secrets or internal hostnames.

## Regression tests

Useful tests:

- missing title returns `400`,
- missing content returns `400`,
- unsupported evidence type returns `400`,
- missing assessment returns `404`,
- unauthenticated request returns `401`,
- unauthorized request returns `403` or intentionally `404`,
- repository exception returns controlled `500`,
- `500` response includes a safe message and request ID,
- `500` response does not expose stack traces or internal details.

## Frontend takeaway

Frontend validation improves UX, but it is not a security boundary. The backend must validate the request, enforce access control and preserve safe state when something unexpected happens.
