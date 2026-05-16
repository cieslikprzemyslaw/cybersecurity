# Lab 02 - CSRF Token Not Tied to User Session

Topic: Broken CSRF token validation  
Focus: testing whether a valid token belongs to the same user session

## Goal

Use a CSRF attack in a legal lab environment to change the victim's email address even though the application uses CSRF tokens.

## Input I Controlled

The controlled inputs were:

- the `email` parameter,
- the `csrf` parameter,
- the external HTML form hosted on the exploit server.

## What Happened

The application included a CSRF token in the email change request:

```http
POST /my-account/change-email
Cookie: session=...

email=test@example.com&csrf=TOKEN
```

At first this looked safer than the no-defence lab.

I used the two provided accounts to test whether the token was tied to the session:

- `wiener:peter`
- `carlos:montoya`

A fresh unused token from one account could be accepted in a request using another account's session.

One useful signal was that the application returned a business logic response such as "email already in use" instead of `Invalid CSRF token`. That showed the CSRF token had passed validation and the request had reached the email update logic.

After confirming this, I used the exploit server to submit a form containing:

```html
<input type="hidden" name="email" value="unique-email@example.com">
<input type="hidden" name="csrf" value="FRESH_UNUSED_TOKEN_FROM_ANOTHER_SESSION">
```

The lab was solved after using a fresh token and a unique email value.

## What I Learned

A CSRF token existing in the request is not enough.

The important question is:

> Is this token tied to the same authenticated session that is making the request?

If the server only checks that the token is valid, but does not check that it belongs to the user's session, a token from one account may be reused against another account.

I also learned to debug response signals:

- `Invalid CSRF token` means token validation failed,
- `302 /login` usually means the session cookie is missing or expired,
- `email already in use` may mean CSRF validation passed and the request reached business logic.

## Impact

In a real application, an attacker could use their own valid token to forge a request against another authenticated user, as long as the token is accepted across sessions.

This can allow unwanted account changes even when the application appears to have CSRF protection.

## Remediation Summary

See `../csrf-samesite-overview.md` for the general remediation section.

Specific to this lab:

- bind CSRF tokens to the user's session,
- reject tokens generated for a different session,
- use unpredictable tokens,
- rotate or invalidate tokens safely,
- test cross-session token reuse during security review.

## Main Takeaway

A CSRF token must be more than present. It must be unpredictable, validated server-side, and tied to the user's session.
