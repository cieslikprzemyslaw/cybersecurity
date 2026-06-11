# Software and Artefact Integrity

## Software updates

An update should not be installed merely because it was downloaded from a familiar URL.

Review:

- source authenticity,
- digital signature,
- trusted verification key,
- version and rollback rules,
- expiry or freshness where applicable,
- verification before installation,
- fail-closed behaviour,
- logging of failed checks.

## Build and deployment artefacts

The artefact built in CI should remain the artefact approved for deployment.

Questions:

- Who can trigger or modify the build?
- Who can replace an artefact after build?
- Is provenance recorded?
- Is the artefact signed?
- Is the signature checked before deployment?
- Can untrusted jobs access signing keys or production secrets?
- Are build and deployment permissions separated?
- Is the deployed digest the approved digest?

This is A08 awareness, not a complete CI/CD security project.

## Third-party browser scripts

A third-party script executes in the browser context of the application and may interact with:

- the DOM,
- forms,
- API responses visible to JavaScript,
- storage accessible to JavaScript,
- user actions,
- application logic.

Loading the script means extending trust to its source and delivery path.

Review:

- Is the provider approved?
- Is the URL controlled?
- Is the version pinned?
- Is the content static enough for SRI?
- Is a restrictive Content Security Policy used?
- What data can the script access?
- What is the fallback if the provider is compromised or unavailable?

## Subresource Integrity

SRI lets the browser compare a fetched resource with an expected cryptographic hash:

```html
<script
  src="https://cdn.example/library.js"
  integrity="sha384-EXPECTED_HASH"
  crossorigin="anonymous">
</script>
```

If the downloaded bytes do not match, the browser blocks the resource.

SRI helps with static third-party resources, but it does not:

- verify npm packages during build,
- replace supplier review,
- protect dynamic scripts whose contents intentionally change,
- prove that the approved version was safe,
- protect a signing or review process that approved a malicious hash.

## Frontend build artefacts

Useful checks include:

- reproducible installation with lockfiles and `npm ci`,
- recorded artefact digests,
- immutable artefact storage,
- promotion of the same artefact between environments,
- restricted write access,
- provenance or signing appropriate to risk,
- verification before deployment.

A pinned version improves reproducibility. It does not alone prove that the downloaded bytes are authentic and unmodified.

## Client-controlled configuration and state

Treat these as untrusted unless verified by a trusted component:

- cookies,
- hidden inputs,
- URL parameters,
- `localStorage`,
- `sessionStorage`,
- JSON configuration downloaded from an editable location,
- feature flags supplied by the browser,
- role or permission values,
- product prices,
- account identifiers,
- CMS configuration rendered into the frontend.

The server should derive sensitive decisions from trusted server-side state.
