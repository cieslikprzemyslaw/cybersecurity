# Lab 01 - Username Enumeration via Different Responses

Topic: Authentication Bypass / Username Enumeration  
Focus: Obvious difference in login error responses

## Goal

The goal of this lab was to understand how different login error messages can reveal whether a username exists.

## Input I Controlled

I controlled the login request body:

```text
username
password
```

The request was sent to the login endpoint using a POST request with form-encoded data.

## What Happened

A login attempt with a random username and password returned an error message saying that the username was invalid.

When testing different usernames with the same wrong password, one username produced a different message indicating that the password was incorrect.

This difference showed that the application treated one username as valid.

Important response signals:

- HTTP status was still `200 OK` for failed attempts,
- the response body contained different error messages,
- response length changed slightly,
- a successful login later produced a `302` redirect to the account page,
- the successful response also set a new session cookie.

## What I Learned

I learned that username enumeration can be very simple when the application gives different messages for invalid usernames and incorrect passwords.

The important lesson was to compare responses carefully and change only one parameter at a time.

First:

```text
username = variable
password = fixed wrong password
```

Then, after identifying a valid username:

```text
username = known candidate
password = variable
```

## Impact

In a real application, this issue can help attackers build a list of valid usernames. They can then use that list for password spraying, credential stuffing, phishing, or targeted brute-force attempts.

The issue does not reveal the password directly, but it reduces the attacker's work.

## Remediation Summary

See `../overview.md` for the general remediation section.

Lab-specific remediation:

- use one generic error message for all failed login attempts,
- avoid messages such as `Invalid username` or `Incorrect password`,
- keep failed-login status codes consistent,
- avoid different response lengths where possible,
- test login responses using valid and invalid usernames,
- add rate limiting and monitoring for repeated failed attempts.

## Main Takeaway

Login error messages must not reveal whether the username or the password was the incorrect part.
