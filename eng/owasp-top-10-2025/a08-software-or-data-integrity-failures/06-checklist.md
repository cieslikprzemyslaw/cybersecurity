# A08 Review Checklist

Use only the sections relevant to the feature being reviewed.

## Asset and trust boundary

- [ ] Identify the software, update, build, configuration, cookie, object, or data being protected.
- [ ] Record its source.
- [ ] Record how it crosses into the trusted component.
- [ ] Identify who can modify it in transit and at rest.
- [ ] Identify which component treats it as authoritative.

## Integrity and authenticity

- [ ] Determine whether integrity, authenticity, confidentiality, or multiple properties are required.
- [ ] Check whether verification uses a trusted key or trusted expected hash.
- [ ] Check whether an attacker can replace both the artefact and its checksum.
- [ ] Confirm verification happens before use, execution, installation, deployment, parsing, or deserialization.
- [ ] Confirm failure causes rejection rather than a fallback to unverified processing.
- [ ] Confirm failed checks are logged without exposing secrets.

## Client-controlled state

- [ ] Review cookies, hidden fields, storage, URL parameters, and frontend-generated values.
- [ ] Check whether roles, permissions, prices, identity, or account state come from the browser.
- [ ] Confirm the backend derives security-sensitive state from a trusted server-side source.
- [ ] Confirm protected endpoints enforce authorization independently of UI visibility.
- [ ] Check whether signed state has expiry, context binding, and replay protection where required.

## Deserialization

- [ ] Identify native object deserialization of user-controlled input.
- [ ] Identify encoding layers such as URL encoding or Base64.
- [ ] Do not treat encoding as integrity protection.
- [ ] Determine object type, fields, and security-sensitive attributes.
- [ ] Confirm integrity checks happen before deserialization.
- [ ] Prefer simple formats and explicit schemas.
- [ ] Reject unexpected types and fields.
- [ ] Do not rely only on validation after deserialization.
- [ ] Do not claim gadget-chain execution or RCE without evidence.

## Third-party scripts and CDNs

- [ ] Inventory third-party browser scripts.
- [ ] Confirm sources are approved.
- [ ] Pin versions where appropriate.
- [ ] Use SRI for suitable static cross-origin resources.
- [ ] Review CSP and script-loading restrictions.
- [ ] Review what application and user data each script can access.
- [ ] Define a response plan for provider compromise.

## Updates, builds, and deployment

- [ ] Verify updates using a trusted signature before installation.
- [ ] Restrict who can publish or approve updates.
- [ ] Protect signing keys from untrusted jobs.
- [ ] Record artefact provenance or digest.
- [ ] Prevent modification between build and deployment.
- [ ] Separate build and deployment permissions.
- [ ] Deploy the same approved artefact between environments where possible.
- [ ] Reject unsigned, unexpected, or altered artefacts.

## Evidence and reporting

- [ ] Record the original artefact or value.
- [ ] Record the single controlled modification.
- [ ] Record whether the application accepted or rejected it.
- [ ] Separate UI evidence from protected endpoint evidence.
- [ ] State the demonstrated impact only.
- [ ] Separate facts from assumptions.
- [ ] Describe the missing integrity or authenticity control.
- [ ] Write a testable security requirement.
