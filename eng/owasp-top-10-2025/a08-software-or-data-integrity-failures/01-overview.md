# A08 Overview

## Definition

Software or Data Integrity Failures occur when an application treats software, code, configuration, updates, artefacts, serialized objects, or critical data as trusted without adequately verifying:

- where the artefact came from,
- whether it was modified,
- whether it is authentic,
- whether it is still valid,
- whether it can be replayed,
- whether the sender was authorised to create or change it.

The failure happens at a trust boundary:

```text
untrusted or externally supplied artefact
    -> missing or insufficient verification
    -> trusted processing
```

## Typical examples

- unsigned software or firmware updates,
- deployment artefacts used without signature or provenance verification,
- CI/CD jobs pulling artefacts from untrusted locations,
- third-party JavaScript loaded from an external CDN without appropriate controls,
- client-controlled cookies treated as trusted session or authorization state,
- serialized objects received from users and deserialized,
- critical configuration or feature flags accepted without integrity protection,
- role, price, account, path, or permission values supplied by the browser and accepted as authoritative.

## What A08 is not

A08 does not mean that every editable cookie, JSON object, or browser value is automatically vulnerable.

The vulnerability exists when:

1. the value crosses into a trusted component,
2. the application treats it as authoritative,
3. integrity or authenticity verification is missing or ineffective,
4. security-relevant behaviour changes because the altered value was accepted.

Not every modified JSON request is insecure deserialization. Insecure deserialization specifically involves reconstructing objects or data structures from user-controllable serialized data and then trusting or executing behaviour associated with that reconstruction.

## Impact

The demonstrated or potential impact depends on the artefact and how it is used:

- privilege escalation,
- unauthorized administrative actions,
- arbitrary file access,
- execution of malicious code,
- deployment of a modified build,
- use of unsafe configuration,
- denial of service,
- replay of previously valid data.

Impact must be based on evidence. A deserialization weakness does not automatically prove remote code execution.

## Primary prevention

- Avoid native object deserialization for untrusted input.
- Prefer simple formats with explicit schemas.
- Reconstruct only expected types and fields.
- Store sensitive authority server-side.
- Verify signatures or MACs before processing trusted client-side state.
- Verify software updates and build artefacts before execution or deployment.
- Use approved sources, repositories, and origins.
- Fail closed when integrity verification fails.
- Apply endpoint-level authorization independently of client-side state.

## Interview-style explanation

> A08 is about crossing a trust boundary without proving that software or data is authentic and unmodified. As a frontend developer, I would look for browser-controlled roles, prices, session properties, feature flags, third-party scripts, and build artefacts that a trusted backend or deployment process accepts without sufficient integrity verification.
