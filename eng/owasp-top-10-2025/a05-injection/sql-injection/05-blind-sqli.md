# Blind SQL Injection

## What it is

Blind SQL Injection happens when the application is vulnerable to SQL Injection, but database data is not returned directly in the response.

Instead, the attacker infers information from response differences or timing.

## Boolean-based blind SQLi

Boolean-based blind SQLi uses true and false conditions.

Example mental model:

```text
condition true  -> response contains marker
condition false -> response does not contain marker
```

In the PortSwigger lab, the marker was:

```text
Welcome back
```

This worked as a true/false oracle:

```text
Welcome back = TRUE
No Welcome back = FALSE
```

## Why it is harder than UNION SQLi

In UNION-based SQLi, the data can appear directly in the response.

In blind SQLi, the attacker cannot see the data directly. Instead, they ask the database many yes/no questions.

Example:

```text
Does the administrator password have length 20?
Is the first character of the password "a"?
Is the second character of the password "b"?
```

## Lab flow

The PortSwigger blind lab used the `TrackingId` cookie.

Evidence flow:

```text
Normal TrackingId -> Welcome back
TRUE condition -> Welcome back
FALSE condition -> no Welcome back
users table exists -> Welcome back
administrator user exists -> Welcome back
password length = 20 -> Welcome back
characters extracted one by one using Intruder
```

## Important learning

Blind SQLi is not about one magic payload. It is about building a reliable oracle and using it carefully.

## Impact

Boolean-based blind SQLi can still lead to sensitive data disclosure and account takeover, even when the application does not display database errors or query results directly.

## Developer fix

- Use prepared statements / parameterized queries.
- Avoid unsafe raw SQL construction.
- Use safe ORM patterns.
- Apply least privilege for database users.
- Add regression tests for true/false payloads.
