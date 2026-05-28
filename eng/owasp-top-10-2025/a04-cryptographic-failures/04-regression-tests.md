# A04:2025 - Cryptographic Failures Regression Tests

## Purpose

Regression tests for Cryptographic Failures should verify that sensitive data and trusted tokens are not protected by weak, predictable or reversible mechanisms.

For these labs, the main focus is remember-me cookie design.

## Test Area 1: Remember-Me Cookie Must Not Reveal Predictable Structure

### Negative Test: Cookie Does Not Decode to Username and Hash

**Given** a user logs in with “remember me” enabled  
**When** the application issues a persistent login cookie  
**Then** decoding the cookie with Base64 should not reveal:

```text
username
password hash
MD5 hash
email
role
user ID
username:hash
```

### Expected Result

The cookie value should look like a random, opaque token.

It should not have a meaningful structure that can be recreated by an attacker.

## Test Area 2: Generated Base64(username:MD5(password)) Cookie Must Fail

### Negative Test: Predictable Candidate Cookie Is Rejected

**Given** an attacker knows the old weak format:

```text
Base64(username:MD5(password))
```

**When** the attacker sends a generated cookie such as:

```http
Cookie: stay-logged-in=<base64-username-md5-candidate>
```

**Then** the application must not authenticate the user.

### Expected Result

The response should not contain:

```html
<p>Your username is: carlos</p>
```

The request should result in a login prompt, invalid token response, or unauthenticated state.

## Test Area 3: Cookie Flags

### Positive Test: Remember-Me Cookie Uses Secure Flags

**Given** a remember-me cookie is issued  
**Then** the cookie should include:

```text
HttpOnly
Secure
SameSite=<appropriate-value>
```

### Notes

- `HttpOnly` prevents JavaScript from reading the cookie through `document.cookie`.
- `Secure` ensures the cookie is only sent over HTTPS.
- `SameSite` should match the application’s legitimate cross-site needs.

## Test Area 4: XSS Cannot Read Sensitive Cookie

### Negative Test: JavaScript Cannot Read Remember-Me Cookie

**Given** a page executes JavaScript  
**When** the script reads:

```js
document.cookie
```

**Then** the remember-me cookie should not appear in the output if JavaScript does not need access to it.

This does not replace fixing XSS, but it reduces cookie theft impact.

## Test Area 5: Token Is Random and Server-Side Validated

### Positive Test: Token Is Opaque

**Given** two users log in with remember-me enabled  
**Then** their remember-me cookies should not reveal user identity or password-derived values.

### Positive Test: Token Is Stored Server-Side Safely

The server should validate the token against server-side storage.

Recommended behaviour:

- store a hash of the token server-side,
- link token to user/session/device,
- track expiry,
- allow revocation,
- rotate token where appropriate.

## Test Area 6: Token Expiry and Revocation

### Positive Test: Expired Token Is Rejected

**Given** a remember-me token has expired  
**When** it is submitted  
**Then** the application should reject it.

### Positive Test: Logout Revokes Token

**Given** a user logs out  
**When** the old remember-me token is used again  
**Then** the token should not authenticate the user.

### Positive Test: Password Change Revokes Old Tokens

**Given** a user changes their password  
**When** an old remember-me token is used  
**Then** the old token should be rejected.

## Test Area 7: Weak Hashes Are Not Used for Password Protection

### Static or Code Review Checks

Search for password-related use of:

```text
md5
sha1
crypto.createHash('md5')
crypto.createHash('sha1')
```

### Expected Result

Password storage should use a strong adaptive password hashing function such as:

- Argon2,
- bcrypt,
- scrypt,
- PBKDF2 where appropriate.

Password hashes should not be sent to the client or stored in cookies.

## Test Area 8: Intruder-Style Generated Tokens Do Not Work

### Negative Test

Create a small test set of generated weak-format cookies:

```text
Base64(carlos:MD5(password1))
Base64(carlos:MD5(password2))
Base64(carlos:MD5(password3))
```

Send each as:

```http
GET /my-account?id=carlos
Cookie: stay-logged-in=<generated-cookie>
```

### Expected Result

All generated weak-format cookies should fail.

No response should authenticate the requester as Carlos.

## Test Area 9: No Sensitive Data in Browser Storage

### Manual Verification

Check:

- cookies,
- localStorage,
- sessionStorage,
- JavaScript globals,
- source maps,
- browser console output.

### Expected Result

These should not contain:

- password hashes,
- raw secrets,
- long-lived sensitive tokens readable by JavaScript,
- predictable authentication tokens.

## What Should Fail After the Fix

After remediation, these should fail:

```text
Base64 decoding a remember-me cookie into username:hash
Using Base64(username:MD5(password)) to authenticate
Reading the remember-me cookie through document.cookie
Using an old remember-me token after logout
Using an old remember-me token after password change
```

## What Should Continue to Work After the Fix

These should still work:

```text
Normal login
Legitimate remember-me login with a valid random token
Logout
Password change
Token expiry
Token revocation
```

## Acceptance Criteria

A fix can be accepted when:

- remember-me tokens are random and opaque,
- cookies do not contain password-derived values,
- Base64 decoding does not reveal sensitive structure,
- generated `username:MD5(password)` cookies fail,
- cookies use `HttpOnly`, `Secure` and appropriate `SameSite`,
- tokens expire,
- tokens can be revoked,
- password changes invalidate old persistent login tokens,
- no password hashes are exposed to the browser.

## Summary

A secure fix should not only change the encoding.

The design must change from:

```text
predictable client-side token based on username and password hash
```

to:

```text
random server-side validated token with expiry, revocation and secure cookie flags
```
