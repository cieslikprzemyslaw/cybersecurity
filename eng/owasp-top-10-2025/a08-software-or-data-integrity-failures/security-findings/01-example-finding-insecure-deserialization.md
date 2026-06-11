# Client-Controlled Serialized Session State Enables Privilege Escalation

## Severity

**High**

## Category

- OWASP Top 10 2025: A08 Software or Data Integrity Failures
- Related impact: Broken Access Control / privilege escalation
- Relevant weakness patterns: deserialization of untrusted data and reliance on cookies without integrity checking

## Summary

The application stores a serialized `User` object in a client-controlled session cookie. The object contains an `admin` boolean that the backend trusts when authorizing access to administrative functionality.

The cookie is URL- and Base64-encoded but has no effective integrity protection visible in the tested flow. A normal user can decode the object, change `admin` from `false` to `true`, re-encode it, and obtain administrative privileges.

## Affected data flow

```text
browser-controlled session cookie
    -> URL decoding
    -> Base64 decoding
    -> PHP object deserialization
    -> trusted admin property
    -> authorization decision
```

## Evidence

Original decoded object:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

Modified object:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

After submitting the modified cookie:

1. the response exposed the `/admin` link,
2. `GET /admin` returned the administration panel,
3. `GET /admin/delete?username=carlos` completed and redirected to `/admin`.

## Demonstrated impact

A normal authenticated user could:

- escalate privileges to administrator,
- access administrative functionality,
- delete another user.

Remote code execution was not tested or demonstrated.

## Root cause

The application treats a security-sensitive property from a deserialized client-controlled object as authoritative. It does not sufficiently verify the integrity of the object and does not independently derive or verify administrative permissions using trusted server-side state.

## Security requirement

> Client-controlled session data must not determine user roles or permissions. Administrative authorization must be derived from trusted server-side state and enforced on every protected endpoint.

Additional requirement:

> If any trusted state is stored client-side, its integrity and authenticity must be verified before parsing, deserialization, or use. Invalid, missing, expired, or replayed state must fail closed.

## Recommended remediation

### Primary remediation

- Replace the serialized client-side session object with an opaque, random session identifier.
- Store user identity, role, and permissions in a protected server-side session store or database.
- Enforce authorization on every administrative endpoint.
- Avoid deserializing native objects from untrusted clients.
- Use a simple schema-based format for any necessary client input.

### Defence in depth

- If client-side state is required, protect it with a strong MAC or digital signature.
- Verify integrity before deserialization or trusted processing.
- Include expiry and relevant context binding.
- Rotate and protect signing keys.
- Log integrity failures and repeated tampering attempts.
- Reject unexpected object types and fields.
- Apply least privilege to the component processing the data.

## Regression tests

1. Modify `admin=false` to `admin=true`; confirm the user remains non-admin.
2. Request `/admin` with altered client state; confirm access is denied.
3. Call an administrative endpoint directly; confirm server-side authorization rejects it.
4. Remove or corrupt the integrity value; confirm fail-closed behaviour.
5. Submit malformed or unexpected serialized objects; confirm controlled rejection before dangerous reconstruction.
6. Replay expired signed state; confirm rejection.
7. Confirm integrity failures are logged without exposing sensitive data.

## Developer takeaway

Base64 and serialization are data representations, not trust controls. Security-sensitive authority must come from trusted server-side state.
