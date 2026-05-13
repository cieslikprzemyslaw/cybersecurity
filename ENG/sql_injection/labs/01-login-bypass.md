# Lab 01 - SQL Injection Login Bypass

> Topic: Authentication logic affected by unsafe SQL query construction  
> Focus: identifying successful login bypass and separating SQLi behaviour from CSRF/session issues

## Goal

Bypass the login form and access the administrator account in a legal lab environment.

## Input I Controlled

The controlled input was the `username` field in the login form.

## What Happened

I tested the login request and used the username field to change the backend SQL logic. The server responded with:

```http
HTTP/2 302 Found
Location: /my-account?id=administrator
Set-Cookie: session=...
```

At first, this did not look like a normal success page because it was a redirect. After using the new session cookie and opening the account page, the application showed that the current user was `administrator`.

## What I Learned

A `302 Found` response can be a success signal during authentication testing. In this case, the redirect and new session cookie were more important than looking for a visible success message in the first response.

I also hit an `Invalid CSRF token` error while manually resending requests. This was a useful reminder that CSRF tokens and session cookies need to match. That error was related to request state, not necessarily to the SQL Injection payload.

## Impact

If this happened in a real application, an attacker could bypass authentication and access an account without knowing the password.

## Remediation Summary

See `../sql-injection-overview.md` for the general remediation section.

Specific to login functionality:

- use parameterized queries for credential lookup,
- never build authentication queries through string concatenation,
- keep CSRF/session protections, but do not treat them as a fix for SQLi,
- add regression tests for malicious input in login fields.

## Main Takeaway

Authentication can fail if user input changes the SQL query used to verify credentials. A redirect, new session cookie, or account page access may confirm success even when the first response does not show a visible success message.
