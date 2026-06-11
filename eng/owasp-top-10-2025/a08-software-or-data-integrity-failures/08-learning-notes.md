# Learning Notes

## Initial understanding

I correctly recognised that browser-controlled data should not be trusted as authoritative. I also understood that Base64 does not provide security.

My initial wording was still too broad:

- I described the artefact only as a cookie rather than a serialized session object inside a cookie.
- I initially mixed checksum properties with digital-signature properties.
- I described encryption as also protecting against tampering without making the need for authentication explicit.

## Corrections

### Cookie versus protected artefact

More precise wording:

> The controlled artefact was a URL- and Base64-encoded serialized PHP `User` object stored in a session cookie.

### Checksum versus signature

Correct distinction:

```text
checksum
    -> does the content match the expected hash?

digital signature
    -> does the content remain unchanged,
       and was it signed by the expected trusted key?

encryption
    -> can an unauthorised party read the content?
```

A checksum does not inherently prove who produced the artefact.

### Encryption and tampering

Encryption primarily provides confidentiality. Integrity and authenticity require a MAC, signature, or authenticated-encryption mechanism.

## TryHackMe learning process

The Python task introduced `pickle` and `__reduce__`.

I attempted to generate a Base64-encoded pickle payload, but the application returned:

```text
UnpicklingError: invalid load key
```

I did not obtain reliable evidence that my own payload executed on the server. I therefore avoid claiming file read or code execution based on that attempt.

The useful lesson was architectural:

```text
untrusted serialized input
    -> pickle deserialization
    -> possible behaviour during reconstruction
```

The primary control is to avoid unpickling untrusted input, not to try to sanitize every dangerous object after reconstruction.

## PortSwigger reasoning

### Observation

The session cookie decoded to:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

### Hypothesis

If the backend trusted the client-controlled `admin` field, changing `b:0` to `b:1` should grant administrative behaviour.

### Controlled test

Only the boolean value was changed. Object names, string lengths, username, and request path initially remained unchanged.

### Evidence

The modified state caused:

1. an admin link to appear,
2. `/admin` to return the administration panel,
3. `/admin/delete?username=carlos` to complete and redirect.

### Conclusion

The backend trusted altered client-controlled object state. This demonstrated privilege escalation and an unauthorized administrative action.

I do not claim remote code execution because no evidence supported it.

## Remediation reasoning

My first idea contained two useful controls:

1. the backend should use its own trusted data rather than accepting frontend state,
2. client-side state should have cryptographic integrity protection if it must be trusted.

The improved design is:

```text
opaque random session ID in cookie
    -> server-side session lookup
    -> role retrieved from trusted session store or database
    -> authorization checked on every protected endpoint
```

Signing client-side data can detect tampering, but it does not remove the need for server-side authorization. Native object deserialization should still be avoided for untrusted input.

## A03 versus A08

I would start with A03 for a malicious npm package because it concerns the broader dependency and supply-chain process.

I would use A08 for:

- an unsigned update,
- an altered build artefact accepted for deployment,
- an unverified third-party script,
- a modified serialized object accepted by the backend.

## Final takeaway

> A08 is not about data being difficult to read. It is about whether a trusted component has evidence that the software or data came from an expected source, remained unmodified, is still valid, and is authorised for the current use.
