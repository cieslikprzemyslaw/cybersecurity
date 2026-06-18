# Practice 02: Transaction and Rollback Review

## Scenario

Evidence file upload normal flow:

```text
1. Create evidence record in the database.
2. Save uploaded file to storage.
3. Update assessment evidence count.
4. Return 201 Created.
```

Failure scenario:

```text
1. Evidence record is created.
2. File is saved.
3. Updating assessment evidence count fails.
4. API returns 500.
```

## What state changed

The failure did not mean that nothing happened.

State that may already have changed:

- an evidence database record may exist,
- an uploaded file may exist,
- the assessment count may remain unchanged,
- the UI may not know whether the evidence was added.

This is a partial state problem:

```text
evidence exists + file exists + assessment count failed
```

## What I corrected

I first focused only on the file storage. The corrected view is wider:

```text
Which parts of the operation already succeeded?
Which part failed?
Can the system return to one known, safe state?
```

For this level, I do not need to design a perfect storage architecture. I need to recognise that an incomplete operation can leave data, files or counters inconsistent.

## A10 learning point

A `500` response does not prove rollback. It only tells the frontend that the API did not return a successful response.

The backend should not leave a normally available evidence item when the full logical operation failed.

## Safe expected behaviour

The backend should either:

- complete the whole logical operation successfully,
- roll back database changes where possible,
- clean up or isolate partially saved files,
- avoid marking the operation as successful when one required step failed,
- return a controlled error,
- and log enough internal evidence for investigation.

At my current level, the simple requirement is:

```text
After a failed evidence upload, the assessment should not contain an active, inconsistent evidence item.
```

## Regression tests

Useful tests:

- simulate failure while updating `evidenceCount`,
- assert that the API returns a controlled `500`,
- assert that the evidence record is not left as active/usable if the full operation failed,
- assert that the uploaded file is not left as a normal available attachment without a valid database record,
- assert that the assessment remains in the previous safe state,
- assert that the error response does not expose internal details,
- assert that a retry does not create duplicate evidence.

## Frontend takeaway

The frontend should not assume that a `500` means nothing was saved. It should avoid showing a confirmed success and may need to refresh the evidence list or show an "unable to confirm" state.
