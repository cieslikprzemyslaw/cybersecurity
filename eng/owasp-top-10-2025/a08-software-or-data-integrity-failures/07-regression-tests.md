# A08 Regression Tests

Select tests relevant to the reviewed feature.

## Serialized client state

### Modified security-sensitive property

Given a valid non-admin session, when `admin=false` is changed to `admin=true`, the user must remain non-admin or the request must be rejected.

Expected:

- no admin UI based solely on the altered state,
- `/admin` returns an authorization failure,
- administrative actions are rejected,
- the integrity failure is logged where appropriate.

### Modified identity or account field

Change a username, user ID, tenant ID, or account reference inside client-controlled state.

Expected:

- the server ignores or rejects the altered authority claim,
- the session remains bound to the original trusted identity.

### Missing or invalid integrity value

Remove or corrupt the MAC/signature.

Expected:

- fail closed,
- no deserialization or trusted processing,
- no fallback to unsigned state.

### Unexpected type or field

Submit:

- an unexpected object type,
- extra object fields,
- malformed serialized data,
- a field with the wrong primitive type.

Expected:

- controlled rejection,
- no dangerous object construction,
- no verbose stack trace,
- no state change.

### Replay

Reuse an expired or previously consumed signed object.

Expected:

- rejection based on expiry, nonce, session, action, or server-side revocation.

## Software and build artefacts

### Modified artefact after build

Change the artefact after it was approved.

Expected:

- digest, signature, or provenance verification fails,
- deployment stops.

### Missing signature

Provide an unsigned update or build where signatures are required.

Expected:

- installation or deployment is blocked.

### Untrusted source

Supply an artefact from an unapproved repository, bucket, registry, or URL.

Expected:

- the artefact is rejected before execution or deployment.

### Corrupted checksum

Change the file while preserving the original expected hash.

Expected:

- mismatch detected,
- operation stops,
- failure is logged.

### Rollback or replay

Attempt to install an old but correctly signed version where rollback is prohibited.

Expected:

- version policy rejects it.

## Third-party browser resources

### SRI mismatch

Change the CDN-hosted script without updating the `integrity` value.

Expected:

- browser blocks execution,
- application handles the missing dependency safely.

### Removed SRI

Remove the `integrity` attribute where policy requires it.

Expected:

- CI, linting, review checks, or CSP policy detect the regression.

### Unapproved origin

Change the script origin.

Expected:

- CSP or application policy prevents loading.

## Authorization defence in depth

Even with a correctly signed client value, protected endpoints must test authorization using trusted server-side state.

Regression:

1. Force the UI to display an admin action.
2. Call the endpoint as a normal user.
3. Confirm the server rejects the action.

This proves that client integrity controls are not the only authorization boundary.
