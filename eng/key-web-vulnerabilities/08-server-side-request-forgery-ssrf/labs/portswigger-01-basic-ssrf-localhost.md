# PortSwigger Lab 01 - Basic SSRF Against the Local Server

## Lab

PortSwigger Web Security Academy:

```text
Basic SSRF against the local server
```

## What I Tested

I tested a stock check feature where the frontend sent a `stockApi` parameter to the backend.

The normal request looked like a standard product stock lookup:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

The normal response returned a stock number.

This suggested that the backend was fetching the URL supplied in `stockApi` and returning the result to the user.

## What I Found

Changing the `stockApi` value to a localhost URL caused the backend to request an internal admin page:

```http
stockApi=http://localhost/admin
```

The response returned the HTML of the admin panel.

This confirmed regular / non-blind SSRF because the backend response was visible in the frontend response.

The admin panel contained a user deletion link for `carlos`.

The state-changing action worked only when requested through the vulnerable `stockApi` parameter, because the backend made the request from the server-side context.

## Important Mistake / Learning Moment

At first, I tried to request the admin delete endpoint directly:

```http
/admin/delete?username=carlos
```

This returned an authorization error because the request came from my browser/user context.

The important lesson was:

> The delete request had to be made through the vulnerable backend request parameter, not directly from my browser.

Correct mental model:

```text
Wrong:
browser -> /admin/delete

Right:
browser -> /product/stock -> backend -> http://localhost/admin/delete
```

## Why It Matters

The admin panel was not safely protected. It trusted requests coming from localhost.

SSRF allowed an external user to make the backend send a request to its own localhost interface.

This bypassed the intended access restriction.

## Root Cause

The root cause was a combination of:

- user-controlled backend request destination,
- backend fetching arbitrary URLs from `stockApi`,
- internal admin functionality trusting localhost requests,
- lack of proper destination validation,
- lack of proper authentication on the internal admin action.

## Impact

Possible impact:

- access to localhost-only admin interface,
- unauthorized administrative actions,
- deletion of users,
- bypass of network/location-based access controls,
- exposure of internal functionality.

## AppSec Principle Violated

The application violated a core trust boundary:

> User-controlled input should not define arbitrary server-side network requests.

It also relied on unsafe trust:

> A request from localhost is not automatically safe.

## Remediation

Safer implementation should:

- avoid accepting arbitrary URLs in `stockApi`,
- use server-side mappings for known stock services,
- allow only expected stock API destinations,
- block localhost, loopback, private, link-local and metadata IP ranges,
- validate the final resolved destination,
- disable or validate redirects,
- require real authentication and authorization for admin actions,
- avoid relying on localhost/internal network location as an access control.

## Regression Test Idea

Add tests to ensure the stock check endpoint rejects:

```text
http://localhost/admin
http://127.0.0.1/admin
http://127.1/admin
http://[::1]/admin
```

Expected result:

```text
The backend must not request internal admin endpoints through stockApi.
```

## Developer Takeaway

If a backend feature fetches a URL provided by the user, that feature can become a bridge into internal systems.

Internal admin panels must not trust localhost alone, because SSRF can make attacker-controlled requests appear to come from localhost.
