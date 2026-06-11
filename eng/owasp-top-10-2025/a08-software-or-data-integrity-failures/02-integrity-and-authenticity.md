# Integrity and Authenticity

## Integrity

Integrity means that software or data has not been changed without authorisation.

Main question:

> Is this the exact content that the trusted component expected?

## Authenticity

Authenticity means that software or data came from the expected source.

Main question:

> Was this produced or approved by the expected identity or signing key?

## Confidentiality

Confidentiality prevents unauthorised disclosure.

Main question:

> Can an unauthorised party read the data?

These properties are related but different.

## Encoding is not security

Base64, URL encoding, and serialization change representation. They do not provide:

- integrity,
- authenticity,
- confidentiality,
- authorization.

Example:

```text
admin=false
    -> Base64
    -> readable and reversible representation
```

An attacker can decode, modify, and re-encode it.

## Checksum

A checksum or hash can detect whether content differs from an expected value:

```text
SHA-256(file) == expected hash
```

A plain checksum does not always prove who created the file. If an attacker can replace both the file and the published hash, the pair can still match.

## Digital signature

A digital signature can verify:

- integrity,
- authenticity of the signing source,

provided that:

- the verification key is trusted,
- the private signing key is protected,
- the verification is correctly implemented,
- the verification happens before use.

Simplified flow:

```text
artefact signed with private key
    -> application verifies using trusted public key
    -> accept or reject before use
```

## Encryption

Encryption primarily protects confidentiality.

It does not automatically guarantee integrity or tamper protection. That requires an authenticated mechanism, such as:

- a MAC or HMAC,
- a digital signature,
- authenticated encryption such as an AEAD mode.

Safe statement:

> Encryption makes data unreadable to unauthorised parties. Integrity and authenticity require an authenticated mechanism.

## Replay

A correctly signed value may still be unsafe if it can be reused outside its intended time or context.

Relevant controls may include:

- expiry,
- issue time,
- nonce,
- audience,
- action binding,
- session binding,
- one-time use,
- server-side revocation.

## Review questions

- What exact property is required: confidentiality, integrity, authenticity, or all three?
- Where does the expected hash or verification key come from?
- Can the attacker replace both the artefact and its checksum?
- Is verification performed before processing?
- Does verification fail closed?
- Can an old but valid signed value be replayed?
