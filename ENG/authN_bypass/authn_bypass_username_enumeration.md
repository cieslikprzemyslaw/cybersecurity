# Authentication Bypass & Username Enumeration — Learning Summary

> AppSec learning note from my transition from Frontend Engineering into Application Security.  
> Based on legal training labs from TryHackMe and PortSwigger Web Security Academy.  
> This write-up avoids flags, real credentials, and full step-by-step answer keys.

---

## Overview

This note summarises what I learned from authentication-focused labs on:

- TryHackMe: Authentication Bypass
- PortSwigger Web Security Academy: Username enumeration via different responses

The main focus was understanding how authentication logic can fail and how small differences in server responses can leak useful information.

The goal was not only to complete the labs, but to understand:

- what authentication bypass means,
- how username enumeration works,
- how tools like Burp Suite, Repeater, Intruder, and ffuf support testing,
- why backend-side authentication checks are essential,
- how developers can prevent these issues.

---

## Key Concepts

### Authentication / AuthN

Authentication answers the question:

> Who are you?

Examples of authentication mechanisms include:

- username and password,
- session cookies,
- JWTs,
- API tokens,
- MFA,
- password reset flows.

An authentication issue happens when the application incorrectly accepts or trusts a user's identity.

---

### Authorization / AuthZ

Authorization answers the question:

> What are you allowed to do?

This is different from authentication.

A user can be correctly logged in, but still should not be allowed to access another user's data or admin functionality.

---

### Authentication Bypass

Authentication bypass happens when an application treats a user as authenticated even though they should not have passed the authentication process.

This can happen because of:

- weak login logic,
- insecure password reset flows,
- predictable or badly validated tokens,
- trusting client-side values,
- issuing sessions incorrectly,
- protected endpoints not checking session state,
- inconsistent backend logic.

---

### Username Enumeration

Username enumeration happens when an application reveals whether a username or email exists.

This can happen through differences in:

- error messages,
- response length,
- response body,
- status codes,
- redirects,
- timing,
- headers,
- cookies.

Example of a risky pattern:

```text
Invalid username
Incorrect password
```

These two different messages reveal that one username does not exist, while another username exists but has the wrong password.

A safer pattern is:

```text
Invalid username or password
```

---

## What I Tested

Across the labs, I looked at:

- login requests,
- password reset requests,
- username parameters,
- password parameters,
- query string values,
- response differences,
- response length,
- cookies and session behaviour,
- manual confirmation after automated testing.

Sanitised request example:

```http
POST /login HTTP/2
Content-Type: application/x-www-form-urlencoded

username=test&password=test
```

---

## Tools Used

### Burp Suite

Used to inspect and understand HTTP requests and responses.

I used it to identify:

- the login endpoint,
- request method,
- request body,
- cookies,
- server responses,
- error messages.

### Burp Repeater

Used for manually resending modified requests and checking how the server responded.

This helped confirm whether the backend response changed depending on the submitted values.

### Burp Intruder

Used in the PortSwigger lab to test multiple username and password values and compare response differences.

Important columns to observe:

- Status,
- Length,
- Words,
- Lines,
- redirects,
- response content.

### ffuf

Used in the TryHackMe lab to automate sending multiple values from wordlists.

`FUZZ` acts as a placeholder that gets replaced by values from a wordlist.

Example concept:

```bash
ffuf -u http://target/login -X POST -d "username=FUZZ&password=test" -w users.txt
```

The key skill was not just running ffuf, but understanding why one response was different from the others.

---

## What I Learned

### 1. 200 OK does not always mean success

Some applications return `200 OK` even when the business action fails.

For example:

```json
{
  "success": false
}
```

or:

```json
{
  "created": false
}
```

This means the HTTP request was processed, but the login or account creation did not succeed.

From a security perspective, it is important to check:

- response body,
- cookies,
- redirects,
- session creation,
- whether protected endpoints are accessible.

---

### 2. Small response differences matter

Even when the status code is the same, small differences can leak information.

Useful things to compare:

- error messages,
- response length,
- number of words,
- number of lines,
- redirects,
- cookies,
- response timing.

In the PortSwigger lab, the key issue was that different login responses revealed whether a username existed.

---

### 3. The frontend is not a security boundary

A user is not limited to the normal browser UI.

They can send requests through:

- Burp Suite,
- Postman,
- curl,
- scripts,
- browser DevTools.

Therefore, the backend must enforce authentication logic and must not rely on the frontend to prevent invalid actions.

---

### 4. Authentication and authorization must not be confused

Authentication bypass is primarily an AuthN issue.

It is about the application incorrectly deciding who the user is or whether the user is logged in.

Broken access control and IDOR are usually AuthZ issues.

They are about a logged-in user accessing something they should not be allowed to access.

---

### 5. Tools help, but the analysis matters more

ffuf and Burp Intruder can send many requests quickly, but they do not understand the business logic.

The important part is interpreting the responses:

- Why is one response different?
- Does this reveal a valid username?
- Was a session issued?
- Did the application redirect?
- Is the user actually authenticated?

---

## Security Impact

Username enumeration can help an attacker:

- identify valid accounts,
- focus password guessing on real users,
- perform targeted phishing,
- improve credential stuffing attempts,
- reduce noise during attacks.

Authentication bypass can be much more serious if it allows access to accounts or protected functionality without valid credentials.

---

## OWASP / AppSec Mapping

### OWASP Top 10

Relevant category:

- Identification and Authentication Failures

Potentially related category:

- Broken Access Control, if the issue leads to access to protected resources.

### Fundamental Principles

- Never trust user input.
- Do not reveal unnecessary information.
- Enforce security on the server side.
- Use defense in depth.
- Validate authentication state on every protected endpoint.

---

## Recommended Remediation

Good practices for authentication flows:

- Use generic login error messages.
- Avoid revealing whether a username or email exists.
- Apply rate limiting to login, registration, and password reset.
- Monitor repeated failed login attempts.
- Use MFA for sensitive accounts.
- Use secure, random, single-use password reset tokens.
- Set short expiry times for reset tokens.
- Validate sessions and tokens server-side.
- Do not rely on frontend logic for authentication decisions.
- Ensure protected endpoints require a valid authenticated session.

Example safer login response:

```text
Invalid username or password
```

Example safer password reset response:

```text
If an account exists, we will send further instructions.
```

---

## Developer Takeaway

As a frontend developer moving into AppSec, the biggest lesson is that authentication security is not only about the login screen.

The real security decision happens on the backend.

Even small UI or API response differences can leak information. A secure implementation should avoid unnecessary detail in authentication errors and should always validate session state server-side.

---

## Personal Reflection

This lab helped me understand how attackers think in terms of differences.

Instead of only looking for obvious errors, I started comparing responses carefully:

- same status, different message,
- same status, different length,
- same form, different backend behaviour.

That mindset is important for AppSec work, because many real vulnerabilities are not obvious at first glance. They appear when you compare how the application behaves under slightly different inputs.

---

## Next Topics to Study

- IDOR / Broken Access Control,
- session management,
- password reset security,
- rate limiting,
- MFA bypass patterns,
- secure authentication design,
- OWASP ASVS authentication controls.
