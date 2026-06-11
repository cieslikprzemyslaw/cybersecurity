# A08 Practical Summary

## Completed coverage

| Exercise | Trust boundary | Missing control | Demonstrated result |
|---|---|---|---|
| TryHackMe Python deserialization | Base64 input to Python deserialization | Safe format and avoidance of untrusted native deserialization | Awareness-level understanding; submitted payload was rejected, so no independent execution claim |
| PortSwigger serialized PHP object | Session cookie to trusted backend object | Integrity protection and trusted server-side authorization state | Privilege escalation and deletion of another user |

## Shared lesson

```text
client-controlled or external artefact
    -> encoding does not make it trusted
    -> verify before trusted processing
```

## What I can now review

- client-controlled session properties,
- serialized cookies,
- native object deserialization,
- frontend-generated role and price values,
- unsigned updates at an awareness level,
- build artefact verification at an awareness level,
- third-party browser scripts and SRI,
- checksum versus signature decisions,
- fail-open behaviour after verification failure.

## Current limit

I have not completed:

- advanced deserialization gadget chains,
- Java or .NET exploit chains,
- signing-key architecture,
- full SLSA/provenance implementation,
- full CI/CD security design.

Those are not required for the current PASS level.
