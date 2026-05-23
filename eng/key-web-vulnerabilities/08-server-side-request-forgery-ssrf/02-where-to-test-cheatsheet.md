# SSRF Where to Test Cheatsheet

## Purpose

Use this checklist when looking for SSRF-prone functionality in legal labs, local practice applications, or authorised testing.

The goal is to identify places where user-controlled input may influence a backend request.

## High-Risk Application Features

SSRF is most likely to appear in features that fetch, import, preview, validate, or process remote resources.

Look for features such as:

- stock checkers,
- image fetchers,
- avatar import from URL,
- PDF generators,
- screenshot generators,
- link preview / URL preview,
- webhook configuration,
- callback URL configuration,
- file import from URL,
- document converters,
- feed importers,
- third-party API integrations,
- server-side analytics that process `Referer` headers,
- XML/document parsers that may fetch external resources.

## Parameters Worth Inspecting

Look for parameters that contain a full URL:

```text
url=
uri=
link=
target=
endpoint=
callback=
webhook=
stockApi=
imageUrl=
avatarUrl=
feedUrl=
next=
redirect=
returnUrl=
```

Also look for parameters that contain only part of a URL:

```text
server=
host=
domain=
path=
api=
service=
resource=
```

Important: SSRF does not always appear as a full URL in a query parameter.

Sometimes the application accepts only a host, path, service name, or hidden form value and then builds the final backend URL server-side.

## Request Body Locations

Check SSRF candidates in:

- query string parameters,
- POST form body,
- JSON body,
- XML body,
- multipart form fields,
- hidden form fields,
- headers,
- cookies,
- stored profile/settings values.

Example form body:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.example.internal/product/stock/check?productId=1&storeId=1
```

Example JSON body:

```json
{
  "imageUrl": "https://example.com/avatar.png"
}
```

Example hidden field:

```html
<input type="hidden" name="avatar" value="/images/avatars/default.png">
```

## Hidden Attack Surface

Some SSRF inputs are not visible in the browser URL bar.

Check:

- DevTools Network tab,
- Burp Proxy history,
- hidden form fields,
- JavaScript-generated requests,
- API calls,
- JSON bodies,
- XML documents,
- headers such as `Referer`.

## Questions to Ask

When reviewing a request, ask:

1. Does this parameter contain a URL, host, path, or service name?
2. Does the backend fetch data from this value?
3. Is the response from that backend request returned to the user?
4. Can I change the host?
5. Can I change the path?
6. Can I reach localhost or a private IP in a legal lab?
7. Is there a filter?
8. Is the filter based on a blacklist or a strict allowlist?
9. Does the backend follow redirects?
10. Does the final resolved destination stay within the intended allowed target?

## Common SSRF Patterns

### Full URL Parameter

```http
stockApi=http://stock.example.internal/product/stock/check
```

Risk:

```text
The attacker may replace the full backend destination.
```

### Partial Host Parameter

```http
server=api
```

Backend builds:

```text
https://api.example.internal/product/stock/check
```

Risk:

```text
The attacker may control the hostname or subdomain used by the backend.
```

### Path-Only Parameter

```http
path=/product/stock/check
```

Backend builds:

```text
https://internal-api.example.local/product/stock/check
```

Risk:

```text
The attacker may use path manipulation or redirect paths if validation is weak.
```

### Hidden Form Field

```html
<input type="hidden" name="avatar" value="/images/default.png">
```

Risk:

```text
Hidden fields are still user-controlled. They can be modified in DevTools or Burp.
```

## Frontend Engineer Angle

From a frontend perspective, SSRF often appears behind normal UI features:

- "Check stock",
- "Import image",
- "Generate preview",
- "Fetch URL",
- "Connect webhook",
- "Validate endpoint".

The UI may look harmless, but the security question is server-side:

> What does the backend do with this value?

## Key Takeaway

SSRF testing starts by finding user-controlled values that influence backend requests.

Do not only search for obvious full URLs. Also inspect hosts, paths, hidden fields, JSON values, headers, and service identifiers.
