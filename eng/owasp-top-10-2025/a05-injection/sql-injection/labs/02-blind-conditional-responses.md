# Lab: Blind SQL Injection with Conditional Responses

## Source

PortSwigger Web Security Academy: **Blind SQL injection with conditional responses**

## Goal

Use boolean-based blind SQL Injection to extract the administrator password and log in.

## Vulnerable feature

The vulnerable input was the `TrackingId` cookie.

Controlled input:

```text
Cookie: TrackingId=...
```

## Why this was blind SQLi

The application did not return database data directly in the response.

Instead, it showed different behaviour depending on whether the injected SQL condition was true or false.

## TRUE/FALSE marker

The marker was:

```text
Welcome back
```

Observed behaviour:

```text
Normal TrackingId -> Welcome back
TRUE condition -> Welcome back
FALSE condition -> no Welcome back
```

This created a true/false oracle.

## Testing flow

### 1. Confirm conditional behaviour

A true condition kept the `Welcome back` message.

A false condition removed the `Welcome back` message.

This confirmed boolean-based blind SQLi.

### 2. Confirm the users table

A subquery checked whether the `users` table existed and could return a row.

Result:

```text
Welcome back = yes
```

### 3. Confirm administrator user

A second subquery checked whether the `administrator` user existed.

Result:

```text
Welcome back = yes
```

### 4. Determine password length

A length condition was tested until the response returned `Welcome back`.

Result:

```text
administrator password length = 20
```

### 5. Extract password character by character

Burp Intruder was used to automate true/false checks for each position and character.

Conceptually, each request asked:

```text
Is character N of the administrator password equal to X?
```

If the response contained `Welcome back`, the tested character was correct.

## Evidence summary

```text
TrackingId normal -> Welcome back
TrackingId + TRUE condition -> Welcome back
TrackingId + FALSE condition -> no Welcome back
users table test -> Welcome back
administrator user test -> Welcome back
password length 20 -> Welcome back
character-by-character checks -> password recovered
```

## Real impact

The vulnerability allowed sensitive data extraction without direct database output. The administrator password was recovered using boolean responses, leading to account takeover.

## Mistakes and learning process

- Blind SQLi was harder than UNION-based SQLi because the data was not displayed directly.
- The key was to identify a reliable response marker.
- The process felt slow because each character had to be extracted through true/false checks.
- Burp Intruder helped automate repeated checks.
- The most important lesson was that blind SQLi is about building an oracle, not about one magic payload.

## Developer remediation

- Use prepared statements / parameterized queries.
- Treat cookie values as untrusted input.
- Avoid unsafe raw SQL in session/tracking logic.
- Use safe ORM patterns.
- Use least-privilege database users.
- Avoid response differences that reveal backend query results.
- Add regression tests for true/false SQLi payloads.

## Regression test ideas

- Injecting true and false SQL conditions into `TrackingId` should not change the presence of `Welcome back`.
- Cookie values should be handled as data, not SQL syntax.
- Password length checks should not be possible through response differences.
- Character extraction should not be possible through repeated conditional requests.
