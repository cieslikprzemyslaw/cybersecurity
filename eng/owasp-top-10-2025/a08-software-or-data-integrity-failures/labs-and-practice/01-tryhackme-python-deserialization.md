# TryHackMe - Python Deserialization Awareness

## Source

[OWASP Top 10 2025: Insecure Data Handling](https://tryhackme.com/room/owasptopten2025three)

Focus: Task 4, A08 Software or Data Integrity Failures.

## Goal

Understand why deserializing untrusted Python `pickle` data is dangerous.

## Relevant flow

```text
user-controlled Base64 input
    -> Base64 decoding
    -> pickle deserialization
    -> object reconstruction
    -> possible behaviour during deserialization
```

## What I learned

- Pickling is Python serialization.
- Unpickling reconstructs objects.
- `__reduce__` can influence how an object is reconstructed.
- Dangerous behaviour may occur during deserialization itself.
- Base64 changes representation only.
- Validation after native object deserialization may be too late.
- Untrusted data should not be passed to `pickle.loads()`.

## Attempt and evidence

I generated a Base64-encoded pickle payload intended for the lab, but the application returned:

```text
Deserialization error: UnpicklingError: invalid load key
```

This proves that the submitted data was not accepted as a valid pickle by that execution path. It does not prove code execution or file read.

I therefore record this task as an awareness exercise rather than claiming a successful independent exploit.

## Root risk

```text
untrusted serialized data
    -> unsafe native deserialization
    -> attacker-influenced object reconstruction
```

## Secure design

- Do not accept pickled objects from untrusted clients.
- Use a simple format such as JSON with a strict schema.
- Parse only expected primitive values.
- Reject unexpected fields and types.
- Keep authority and sensitive state server-side.
- Verify any required client state before parsing or reconstructing it.
- Apply least privilege and isolate dangerous processing.

## Regression ideas

- Submit malformed pickle data.
- Submit an unexpected object type.
- Submit extra fields.
- Submit unsigned or modified data.
- Confirm the server rejects the input before object reconstruction.
- Confirm no file, network, command, or state-changing side effect occurs.
