# Insecure Deserialization

## Definitions

Serialization converts an object or complex data structure into a format suitable for storage or transport.

Deserialization reconstructs the object or data structure.

```text
object
    -> serialization
    -> bytes or text
    -> transport/storage
    -> deserialization
    -> reconstructed object
```

In Python, serialization may be called pickling. In Ruby, it may be called marshalling.

## Insecure deserialization

Insecure deserialization occurs when user-controllable serialized data is reconstructed and trusted.

Mental model:

```text
user-controlled serialized data
    -> deserialization
    -> trusted object creation
    -> application behaviour
```

Possible impact includes:

- manipulation of object attributes,
- privilege escalation,
- unexpected object creation,
- invocation of dangerous methods,
- file access,
- denial of service,
- remote code execution.

Only claim impact supported by evidence.

## Why validation after deserialization is weak

Dangerous behaviour can occur during object reconstruction itself. Checking the object only after deserialization may be too late.

This is why the strongest control is:

> Do not deserialize untrusted native objects.

## Recognising serialized data

Possible indicators include:

- readable PHP object notation such as `O:4:"User"...`,
- Java serialized data commonly beginning with a recognisable Base64 signature,
- binary or Base64-encoded blobs in cookies,
- Python pickle data,
- framework-specific session formats,
- object attributes embedded in client-controlled state.

Encoding or unreadability does not make the object trustworthy.

## Review workflow

1. Identify the serialized value.
2. Determine where it comes from.
3. Decode representation layers without assuming they are security controls.
4. Identify object type and fields.
5. Find security-sensitive attributes.
6. Make one controlled change.
7. Re-encode correctly.
8. Observe whether altered state is accepted.
9. Separate visible UI changes from protected endpoint behaviour.
10. Record only demonstrated impact.

## Safer design

- Avoid native deserialization of untrusted data.
- Use JSON or another simple format with an explicit schema.
- Parse into safe primitive types.
- Reject unexpected fields and types.
- Do not let the client provide authoritative roles, permissions, identities, or prices.
- Keep sensitive state server-side.
- If client-side state is necessary, verify integrity before parsing or deserialization.
- Apply endpoint-level authorization using trusted server-side state.
- Use expiry and anti-replay controls where appropriate.
- Log integrity failures.

## Key lesson from the completed lab

The vulnerability was not merely:

> I changed a cookie.

The accurate lesson was:

> The application deserialized a client-controlled PHP object and trusted its modified `admin` attribute as authorization state without sufficient integrity protection or server-side verification.
