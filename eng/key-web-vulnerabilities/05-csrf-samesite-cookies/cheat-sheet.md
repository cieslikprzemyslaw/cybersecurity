# CSRF + SameSite Cookies Practical Cheat Sheet

## Purpose

Use this cheat sheet during authorised testing, code review, AppSec learning, or internal security work.

The goal is not to provide lab-specific answers. The goal is to help you recognise whether a state-changing request is protected properly against CSRF and whether browser cookie behaviour, token validation, or SameSite settings create risk.

Focus especially on actions such as:

- changing an email address,
- changing a password,
- updating profile details,
- changing account settings,
- modifying payment information,
- changing roles or permissions,
- submitting security preferences,
- performing transactions.

## Core Mental Model

CSRF is not about stealing the user's cookie.

CSRF abuses the fact that the browser may automatically attach cookies to requests sent to a site where the user is already authenticated.

A valid session cookie proves that the browser has an authenticated session. It does not prove that the user intentionally performed the action.

## First Questions to Ask

When reviewing or testing a request, ask:

- Does this request change server-side state?
- Is the user authenticated when making this request?
- Does the request rely on cookies for authentication?
- Is there a CSRF token?
- Is the token random and unpredictable?
- Is the token tied to the user's session?
- What happens if the token is removed?
- What happens if the token is changed?
- What happens if a token from another user/session is used?
- Can the request be triggered from an external page?
- What SameSite value is set on the session cookie?
- Does the application support method override?
- Are Origin, Referer, and CORS handled safely?

## Basic Testing Flow

1. Log in to the application.
2. Perform a state-changing action normally.
3. Capture the request in Burp Suite.
4. Identify:
   - HTTP method,
   - endpoint,
   - cookies,
   - request body or query parameters,
   - CSRF token if present,
   - Origin header,
   - Referer header,
   - SameSite cookie attribute.
5. Send the request to Repeater.
6. Test safely in an authorised environment:
   - remove the CSRF token,
   - change the token value,
   - reuse a token from another session,
   - change the HTTP method,
   - try moving body parameters into the query string if relevant,
   - check whether method override is supported,
   - reproduce the request from an external HTML page.

## What to Look for in the Request

A potentially vulnerable request often looks like this:

```http
POST /account/change-email HTTP/2
Host: example.com
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

email=new@example.com
```

Risk indicators:

- no CSRF token,
- token is static,
- token is predictable,
- token is only Base64-encoded user data,
- token can be reused across sessions,
- state-changing action uses `GET`,
- session cookie is sent cross-site,
- application relies only on session cookies,
- Origin or Referer validation is missing or weak,
- CORS allows sensitive credentialed requests from untrusted origins.

## Safe HTML POST Test Template

Use this pattern only in legal labs or authorised testing when checking whether a form-encoded `POST` request can be reproduced from another site.

```html
<html>
  <body>
    <form method="POST" action="https://TARGET.example.com/account/change-email">
      <input type="hidden" name="email" value="new-unique-email@example.com">
    </form>

    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

### What to Replace

- `https://TARGET.example.com/account/change-email`  
  Replace with the real state-changing endpoint.

- `email`  
  Replace with the real parameter name from the request.

- `new-unique-email@example.com`  
  Replace with a unique test value.

### What This Checks

This checks whether the server accepts a state-changing request that is triggered from another site and authenticated only by the victim browser's cookies.

If it works, the application likely does not verify user intent properly.

## POST Form With CSRF Token

Use this pattern when the request requires a CSRF token, but you are testing whether the token is implemented correctly.

```html
<html>
  <body>
    <form method="POST" action="https://TARGET.example.com/account/change-email">
      <input type="hidden" name="email" value="new-unique-email@example.com">
      <input type="hidden" name="csrf" value="FRESH_UNUSED_TOKEN">
    </form>

    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

### Important Checks

Before using this pattern, confirm:

- the token is fresh,
- the token has not already been submitted,
- the token is not tied to the wrong session,
- the test value is unique,
- the server accepts or rejects reused tokens correctly,
- the server rejects tokens from another user's session.

A token being present is not enough. It must be unpredictable and bound to the user's session or request context.

## SameSite Lax Method Override Check

`SameSite=Lax` usually blocks cookies on cross-site `POST` requests, but allows cookies in some top-level `GET` navigations.

If the backend supports method override, a request may look like a `GET` to the browser but be treated as a `POST` by the server.

Example pattern:

```html
<html>
  <body>
    <script>
      document.location = "https://TARGET.example.com/account/change-email?email=new-unique-email@example.com&_method=POST";
    </script>
  </body>
</html>
```

### How to Test the Behaviour

First, check whether normal `GET` is rejected:

```http
GET /account/change-email?email=test@example.com HTTP/2
Host: example.com
```

A response like this suggests the endpoint expects `POST`:

```http
405 Method Not Allowed
Allow: POST
```

Then check whether method override is accepted:

```http
GET /account/change-email?email=test@example.com&_method=POST HTTP/2
Host: example.com
```

If this changes state, the backend is treating the request as `POST` even though the browser made a `GET` navigation.

## SameSite Quick Reference

### SameSite=Strict

Cookies are sent only in a same-site context.

This provides the strongest CSRF protection from the cookie side, but it may break some legitimate cross-site flows such as external login redirects or payment journeys.

### SameSite=Lax

Cookies are sent in same-site requests and some top-level cross-site navigations using safe methods such as `GET`.

This helps against many basic CSRF attacks, especially cross-site `POST`, but it is not a complete defence.

### SameSite=None

Cookies are sent in cross-site requests.

This must be combined with `Secure` and should be used only when cross-site cookies are genuinely required.

If `SameSite=None` is used on sensitive authenticated cookies, strong CSRF protections are especially important.

## Common Response Signals

Useful signals during testing:

- `302 Found` to an account page may indicate the state-changing action succeeded.
- `302 Found` to `/login` usually means the session cookie is missing, expired, or not sent.
- `Invalid CSRF token` means the request failed CSRF validation.
- A business logic error such as `email already in use` may mean the CSRF token was accepted and the request reached application logic.
- `405 Method Not Allowed` means the endpoint does not accept that HTTP method.
- A successful state change after `_method=POST` suggests method override may be enabled.

## Developer Review Checklist

Use this during implementation or code review:

- Are all state-changing endpoints protected?
- Are CSRF tokens generated server-side?
- Are tokens random and unpredictable?
- Are tokens tied to the authenticated session?
- Are tokens validated server-side?
- Are invalid or missing tokens rejected?
- Are tokens regenerated or expired appropriately?
- Are state-changing actions blocked via `GET`?
- Are method override features necessary?
- If method override exists, is it limited and protected?
- Are cookies configured with appropriate `SameSite`, `Secure`, and `HttpOnly` attributes?
- Are `Origin` and `Referer` checks used only as defence-in-depth?
- Is CORS restricted to trusted origins for sensitive actions?
- Are credentialed cross-origin requests avoided unless absolutely required?
- Is XSS prevented, since XSS can often bypass CSRF protections?

## Practical Takeaway

Do not ask only: "Is the user logged in?"

Ask:

> Did the server verify that this authenticated user intentionally submitted this specific state-changing request?

CSRF protection is about proving user intent, not just user authentication.
