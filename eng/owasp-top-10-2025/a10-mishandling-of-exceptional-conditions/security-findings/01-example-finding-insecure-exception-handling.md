# Example Finding: Insecure Exceptional-condition Handling Can Leave Partial Evidence State

## Summary

The evidence upload workflow may leave partial state if one step succeeds and a later step fails. A generic `500` response is returned to the client, but this response alone does not prove that the backend rolled back or cleaned up earlier changes.

## Affected workflow

```text
POST /api/assessments/{assessmentId}/evidence-file
```

Normal flow:

```text
create evidence record
save evidence file
update assessment evidence count
return 201 Created
```

## Vulnerable behaviour

If the assessment summary update fails after the evidence record and file are already created, the API may return a generic `500`.

The response avoids information disclosure, but it does not prove that the evidence record or file was removed.

## Security impact

The application may retain inconsistent state:

- evidence record exists but assessment summary does not reflect it,
- file exists without a complete controlled workflow,
- UI shows failure while backend has partial data,
- retry may create duplicates,
- reports or audit review may use inconsistent data.

Affected properties: integrity, accountability and availability.

## Evidence to collect

- request triggering the failure in a controlled test,
- response status and body,
- database state before and after,
- file storage state before and after,
- assessment summary before and after,
- logs with request/correlation ID.

Do not claim data corruption without verifying changed state.

## Recommendation

- Treat the upload as one logical operation.
- Roll back database changes where possible.
- Clean up or quarantine files when later steps fail.
- Return controlled errors without internal details.
- Log safe internal context and correlation ID.
- Ensure retry does not create duplicate active records.
- Add regression tests for each failing step.

## Regression tests

- Simulate failure after evidence record creation and verify no active partial evidence remains.
- Simulate failure after file save and verify file is removed, quarantined or marked for cleanup.
- Verify controlled `500` without stack trace, SQL query, path or secrets.
- Verify retry does not create duplicate active records.

## Interview-style takeaway

> A generic error response is useful, but it is not proof of safe recovery. During review I would check what state changed before the failure, whether rollback or cleanup happened, whether retry can duplicate data, and which test proves the workflow returns to a known safe state.
