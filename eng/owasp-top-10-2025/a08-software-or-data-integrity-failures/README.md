# A08: Software or Data Integrity Failures

This folder contains my practical notes for OWASP Top 10 2025 - A08: Software or Data Integrity Failures.

## Status

**PASS — Frontend Developer transitioning into AppSec**

Completed work:

- TryHackMe: OWASP Top 10 2025 - Insecure Data Handling, Task 4: A08,
- PortSwigger: Modifying serialized objects,
- review of integrity, authenticity, checksums, digital signatures, encryption, third-party scripts, Subresource Integrity, client-controlled state, and artefact verification,
- comparison of A03 Software Supply Chain Failures with A08 Software or Data Integrity Failures.

The status does not mean expert knowledge of cryptography, CI/CD security, gadget chains, or exploit development. It means I can identify basic integrity trust boundaries, recognise unsafe deserialization, separate facts from assumptions, propose developer-focused controls, and write useful regression tests.

## Core mental model

```text
software or data artefact
    -> trust boundary
    -> integrity and authenticity verification
    -> trusted processing
```

Main review questions:

1. What software, configuration, object, cookie, or data is being trusted?
2. Where did it come from?
3. Who can modify it?
4. How is its integrity verified?
5. How is its source authenticated?
6. Is verification performed before the artefact is used or deserialized?
7. Can an old but valid artefact be replayed?
8. What happens when verification fails?
9. What evidence proves altered data was accepted?
10. What should the secure design use as the source of truth?

## Frontend perspective

The browser is not a trusted authority.

- Base64 is encoding, not security.
- Hidden fields, cookies, `localStorage`, and `sessionStorage` are user-controlled.
- Frontend-generated roles, prices, feature flags, or permissions must not be authoritative.
- Loading a third-party script means trusting code that executes in the application's browser context.
- A pinned version helps reproducibility but does not by itself prove artefact integrity.
- SRI can verify static browser resources against an expected hash, but it is not a universal software supply-chain control.

A vulnerability exists when a trusted component accepts altered or untrusted data as authoritative without sufficient verification. The mere presence of client-controlled data is not automatically a vulnerability.

## Completed practical patterns

### Python pickle awareness

The TryHackMe task demonstrated why Python `pickle` data from an untrusted user is dangerous. Object reconstruction can trigger behaviour during deserialization. My generated payload was rejected with an `UnpicklingError`, so I do not claim independent proof that my payload read the server file. The retained lesson is the unsafe trust boundary: untrusted serialized input must not be passed to native object deserialization.

### Client-controlled PHP session object

The PortSwigger lab stored a serialized PHP `User` object in a URL- and Base64-encoded session cookie:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

Changing only:

```text
b:0
```

to:

```text
b:1
```

caused the application to:

- expose the `/admin` link,
- return the admin panel,
- allow deletion of another user.

The real issue was not simply that a cookie could be edited. The backend deserialized client-controlled state and trusted the altered `admin` property as authorization state.

## Start here

- [Overview](01-overview.md)
- [Integrity and authenticity](02-integrity-and-authenticity.md)
- [A03 versus A08](03-a03-vs-a08.md)
- [Insecure deserialization](04-insecure-deserialization.md)
- [Software and artefact integrity](05-software-and-artifact-integrity.md)
- [Review checklist](06-checklist.md)
- [Regression tests](07-regression-tests.md)
- [Learning notes](08-learning-notes.md)
- [Labs and practice](labs-and-practice/README.md)
- [Example security finding](security-findings/01-example-finding-insecure-deserialization.md)

## Direct learning links

- [OWASP A08:2025 - Software or Data Integrity Failures](https://owasp.org/Top10/2025/A08_2025-Software_or_Data_Integrity_Failures/)
- [TryHackMe - OWASP Top 10 2025: Insecure Data Handling](https://tryhackme.com/room/owasptopten2025three)
- [PortSwigger - Insecure Deserialization](https://portswigger.net/web-security/deserialization)
- [PortSwigger Lab - Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)
