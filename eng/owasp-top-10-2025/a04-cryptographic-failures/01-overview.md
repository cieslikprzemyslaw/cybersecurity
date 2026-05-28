# A04:2025 - Cryptographic Failures

## What This OWASP Category Means

Cryptographic Failures happen when an application fails to protect sensitive data properly.

This can happen because cryptography is missing, weak, misused, outdated, predictable, or exposed through poor token/key handling.

In simple terms:

> A04 is about sensitive data or trusted tokens not being protected strongly enough.

This does not mean that an AppSec engineer needs to invent or implement custom cryptography. In most real reviews, the safer mindset is the opposite:

> Do not build your own crypto. Check whether the application uses well-known, standard, reviewed mechanisms correctly.

## Why It Matters

Applications rely on cryptography and token design for many sensitive behaviours:

- password storage,
- session protection,
- remember-me cookies,
- password reset links,
- email verification links,
- API tokens,
- JWTs,
- signed cookies,
- encryption in transit,
- encryption at rest,
- protection of secrets and keys.

A weak cryptographic design can lead to:

- account takeover,
- password cracking,
- session hijacking,
- sensitive data disclosure,
- token forgery,
- bypassing trust boundaries,
- replay of old tokens,
- exposure of secrets.

## Practical Review Context

For this category, I completed two PortSwigger Web Security Academy labs focused on weak remember-me cookie design:

1. **Offline password cracking**
2. **Brute-forcing a stay-logged-in cookie**

Both labs used a weak persistent-login cookie pattern:

```text
stay-logged-in = Base64(username:MD5(password))
```

The important issue was not only that Base64 was used. The deeper issue was that the cookie contained a predictable password-derived value.

The cookie could be decoded, understood, recreated, and brute-forced.

## Key Concepts I Practised

### Base64 Is Encoding, Not Encryption

Base64 is not encryption.

It does not use a secret key. It is designed to be reversible. Anyone who has the value can decode it and read the original data.

In the lab, decoding the cookie revealed a structure similar to:

```text
username:md5(password)
```

That means the cookie was not encrypted or protected. It was only encoded.

### MD5 Is Not Suitable for Password Protection

MD5 is a fast and outdated hashing algorithm.

It is not suitable for password protection because attackers can test many candidate passwords very quickly and compare the resulting hash with the target hash.

For passwords, modern applications should use adaptive, salted password hashing such as:

- Argon2,
- bcrypt,
- scrypt,
- PBKDF2 where appropriate.

However, the remember-me token should not be based on the password hash at all.

### Remember-Me Token Design

A safer remember-me token should not look like:

```text
Base64(username:MD5(password))
```

A safer design should use a long, random, high-entropy token.

The server should store a hashed version of that token and link it to:

- the user,
- the device/session,
- an expiry time,
- revocation state,
- optional rotation behaviour.

The cookie should be protected with:

- `HttpOnly`,
- `Secure`,
- appropriate `SameSite`,
- an expiry time.

## Common Examples

Cryptographic Failures can include:

- Base64 used as if it were encryption,
- passwords stored or exposed with weak hashes,
- MD5 or SHA1 used for password-related protection,
- predictable remember-me tokens,
- password-derived values stored in cookies,
- reset tokens generated predictably,
- secrets committed to source code,
- weak or reused keys,
- missing TLS or weak TLS configuration,
- sensitive data sent over HTTP,
- missing `Secure` on sensitive cookies,
- missing `HttpOnly` on cookies that should not be readable by JavaScript,
- weak random number generation,
- custom cryptography,
- tokens that never expire or cannot be revoked.

## Common Root Causes

Typical causes include:

- confusing encoding with encryption,
- using old examples or legacy crypto patterns,
- using fast hashes for password-related values,
- storing password-derived data in client-side cookies,
- building custom token formats,
- not using server-side token validation,
- missing secure cookie flags,
- treating the browser as a safe storage location,
- no review of token design,
- lack of expiry, rotation and revocation,
- weak understanding of hashing vs encryption vs signing.

## Impact

The impact depends on the data and token being protected.

In these labs, the impact was account takeover because the remember-me cookie could be used to access another user's account.

Possible real-world impacts include:

- offline password cracking,
- persistent login token forgery,
- account takeover,
- session hijacking,
- replay of stolen tokens,
- exposure of password-derived data,
- easier exploitation when combined with XSS,
- long-lived unauthorized access if remember-me tokens do not expire or cannot be revoked.

## Related Vulnerabilities I Have Already Practised

Related topics from my previous learning:

- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite Proxy and Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md)
- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Broken Access Control / IDOR](../../key-web-vulnerabilities/02-broken-access-control-idor/README.md)
- [Cross-Site Scripting](../../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [Security Misconfiguration](../a02-security-misconfiguration/01-overview.md)

These topics are related because weak crypto/token design often becomes worse when combined with browser-side issues such as XSS, missing cookie flags, or poor session handling.

## My Takeaway as a Frontend Engineer

The most important frontend-facing lesson is:

> Anything stored in the browser must be treated carefully. If JavaScript can read it, XSS can probably steal it.

A frontend developer should understand that:

- Base64 does not protect data,
- sensitive tokens should not be stored in readable JavaScript storage without a strong reason,
- cookies that do not need JavaScript access should usually be `HttpOnly`,
- remember-me tokens should not contain usernames, password hashes or predictable structures,
- token and cookie design is a backend/security decision, but frontend developers often see the symptoms in DevTools and Burp.

## What Good Looks Like

A safer implementation should include:

- random high-entropy remember-me tokens,
- server-side token validation,
- hashed token storage server-side,
- token expiry,
- token revocation,
- token rotation where appropriate,
- no password hash in client-side cookies,
- no predictable token format,
- `HttpOnly` for cookies that JavaScript should not read,
- `Secure` for HTTPS-only cookies,
- appropriate `SameSite`,
- TLS enforced for sensitive flows,
- modern password hashing for actual password storage,
- no custom cryptography.

## Summary

The main A04 lesson from these labs:

> A token that looks encoded or complex is not automatically secure. If the format is predictable and based on weak or password-derived values, it may be possible to decode, recreate, brute-force or steal it.
