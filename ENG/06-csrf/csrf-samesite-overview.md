# CSRF + SameSite Cookies

## TL;DR

Cross-Site Request Forgery (CSRF) is a vulnerability where an attacker tricks an authenticated user's browser into sending a request to an application where the user is already logged in.

The attacker does not need to steal the password or session cookie. The browser may automatically attach the victim's session cookie to requests sent to the target site. If the application accepts the request based only on the cookie, a state-changing action can be performed without the user's intention.

## What I Practised

I practised CSRF using TryHackMe and PortSwigger labs.

The exercises covered:

- finding state-changing requests,
- checking whether requests rely only on session cookies,
- confirming whether a CSRF token exists,
- testing weak or missing CSRF protection,
- understanding why `POST` does not automatically prevent CSRF,
- checking whether tokens are tied to the user's session,
- understanding SameSite cookie behaviour,
- testing a SameSite `Lax` bypass using method override,
- reviewing why CORS, Referer checks, and custom headers are not complete protection by themselves.

## Key Concept

CSRF happens when the server treats an authenticated request as intentional only because it contains a valid session cookie.

A session cookie proves that the browser has an authenticated session. It does not prove that the user intentionally submitted the action.

A CSRF attack usually needs:

- an authenticated victim,
- a state-changing action,
- missing or broken verification of request origin or user intent.

Common CSRF-prone actions include:

- changing an email address,
- changing a password,
- updating account settings,
- editing profile details,
- changing payment information,
- submitting user preferences,
- modifying roles or permissions.

`POST` is not a CSRF defence. A malicious site can still submit a `POST` request using an HTML form. `GET` is even more risky when it changes state because it can be triggered by links, redirects, or images.

CSRF tokens help only when they are strong, unpredictable, validated server-side, and tied to the user's session.

## What the User Controls

In the labs, the attacker controlled:

- form fields such as `email`,
- URL query parameters,
- hidden form inputs such as `csrf`,
- the HTML form hosted on an external exploit server,
- JavaScript used to auto-submit a form or redirect the browser,
- the `_method` parameter used for method override,
- in some scenarios, a predictable or reusable CSRF token.

The attacker did not directly control the victim's session cookie. The browser supplied the cookie automatically when the request was sent to the target application.

## Where the Input Goes

The controlled input was processed by backend state-changing endpoints, mainly:

- `/my-account/change-email`,
- password change functionality in CSRF practice rooms,
- role or account settings update functionality,
- server-side CSRF token validation checks,
- method override handling that converted a top-level `GET` navigation into backend `POST` behaviour.

In the SameSite lab, the important path was:

```http
GET /my-account/change-email?email=...&_method=POST
```

The browser treated the navigation as `GET`, while the backend treated it as a state-changing `POST`.

## Impact

CSRF impact depends on what the vulnerable action can do.

Realistic impact includes:

- changing a user's email address,
- changing a user's password,
- updating account settings,
- changing payment details,
- modifying roles,
- performing unwanted actions as the victim.

CSRF does not always let the attacker read the response because of browser protections such as the Same-Origin Policy. However, reading the response is not always required if the attacker can successfully trigger the action.

## Developer Remediation

Recommended fixes:

- add strong, random, unpredictable CSRF tokens to state-changing requests,
- bind CSRF tokens to the user's session,
- validate tokens server-side on every sensitive action,
- reject missing, empty, reused, predictable, or cross-session tokens,
- do not generate tokens from predictable values such as account numbers, roles, usernames, or Base64-encoded data,
- avoid state-changing actions via `GET`,
- do not treat `POST` as a security control by itself,
- avoid unsafe method override behaviour for sensitive actions,
- set session cookies with appropriate `SameSite` values,
- use `SameSite=Lax` or `SameSite=Strict` where possible,
- use `SameSite=None; Secure` only when cross-site cookies are genuinely required,
- validate `Origin` and/or `Referer` as defence-in-depth, not as the only defence,
- configure CORS carefully and never allow sensitive credentialed actions from untrusted origins,
- fix XSS because XSS can often bypass CSRF protections by reading tokens or making same-origin requests.

## Review Checklist

During testing or code review, ask:

- Does this request change server-side state?
- Does the request rely only on a session cookie?
- Is there a CSRF token?
- Is the token random and unpredictable?
- Is the token tied to the user's session?
- What happens if the token is missing?
- What happens if the token is reused from another user?
- Can a fresh token from one account be used with another account's session?
- Can the request be reproduced from an external HTML form?
- Is a state-changing action reachable via `GET`?
- Does the application support `_method=POST` or similar method override?
- Does method override allow SameSite `Lax` protections to be bypassed?
- What SameSite value is used on the session cookie?
- Are CORS rules safe for sensitive endpoints?
- Is the application relying only on Referer or custom headers?

## Main Takeaway

CSRF is about abusing browser behaviour, not stealing cookies.

A valid session cookie proves authentication, but it does not prove user intent.

The practical lesson from this topic:

> CSRF protection must verify that the request was intentionally made by the authenticated user, not only that the browser sent a valid session cookie.
