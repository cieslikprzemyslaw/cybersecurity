# Lab 02 - Username Enumeration via Subtly Different Responses

Topic: Authentication Bypass / Username Enumeration  
Focus: Tiny response differences in mostly generic login errors

## Goal

The goal of this lab was to understand that username enumeration does not always rely on obvious messages. Even a small difference in the response body can reveal a valid username.

## Input I Controlled

I controlled the login request body:

```text
username
password
```

The request was sent to the login endpoint using a POST request with form-encoded data.

## What Happened

The application displayed a generic-looking error message for failed login attempts.

At first, this looked safer than the previous lab because the message did not clearly say whether the username or password was wrong.

However, when comparing many responses, one response contained a tiny difference in the warning message, such as punctuation or spacing. This showed that the backend was following a different logic path for one username.

Useful response signals:

- HTTP status stayed as `200 OK` for failed attempts,
- the visible error message looked almost the same,
- the raw HTML contained a small difference,
- Burp Grep - Extract helped isolate the warning message,
- a successful login later produced a `302` redirect and a new session cookie.

## What I Learned

I learned that generic messages are not enough if the raw responses are not truly identical.

Small differences matter, including:

- punctuation,
- trailing spaces,
- response length,
- hidden HTML differences,
- different template branches.

I also learned that Burp Grep - Extract is useful for comparing many responses quickly. Instead of reading the full HTML manually, I extracted only the warning message and compared that column in Intruder.

## Impact

In a real application, attackers can automate response comparison and detect small differences that normal users would not notice.

This can still allow them to identify valid usernames and focus further attacks on real accounts.

## Remediation Summary

See `../authn-bypass-username-enumeration-overview.md` for the general remediation section.

Lab-specific remediation:

- use one shared error message constant or template,
- make sure all failed login paths return the same visible message,
- avoid accidental punctuation or spacing differences,
- test raw HTML responses, not only the browser view,
- compare status codes, response length, redirects, cookies, and timing,
- include negative authentication tests in QA/security testing.

## Main Takeaway

A message can look generic in the browser but still leak information through tiny differences in the raw response.
