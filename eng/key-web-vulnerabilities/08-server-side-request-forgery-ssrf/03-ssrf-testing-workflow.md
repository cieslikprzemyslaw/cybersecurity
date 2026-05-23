# SSRF Testing Workflow

## Purpose

Use this workflow for legal labs and authorised testing when investigating potential SSRF behaviour.

The aim is to understand whether user-controlled input can make the backend send requests to unintended destinations.

## Basic Workflow

### 1. Identify a Candidate Parameter

Look for parameters that contain or influence:

- full URLs,
- hostnames,
- paths,
- service names,
- callback URLs,
- webhook URLs,
- image/document URLs.

Example:

```http
stockApi=http://stock.example.internal/product/stock/check?productId=1&storeId=1
```

### 2. Confirm the Normal Behaviour

Send the request normally and record:

- status code,
- response body,
- response length,
- response type,
- expected functionality.

Example baseline:

```text
Normal stock check returns a number such as 350.
```

This baseline is important because SSRF testing relies on comparing differences.

### 3. URL Decode the Input

If the parameter is URL-encoded, decode it to understand the actual backend destination.

Encoded:

```text
http%3A%2F%2Fstock.example.internal%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```

Decoded:

```text
http://stock.example.internal:8080/product/stock/check?productId=1&storeId=1
```

### 4. Test Whether the Backend Fetches the URL

Change the URL in a controlled way and compare the response.

In a legal lab, this might involve testing:

```text
http://localhost/
http://127.0.0.1/
http://127.1/
```

Do not use these tests against real systems without authorisation.

### 5. Check If the Response Is Returned

If the application returns the backend response, this is regular / non-blind SSRF.

Signs:

- HTML from an internal page appears in the response,
- internal admin page content appears,
- status/length/title changes clearly,
- internal error messages appear.

If no response is returned, it may be blind SSRF.

### 6. Identify Whether the Target Is Localhost or Another Backend

SSRF may target:

```text
localhost / 127.0.0.1
```

or another internal system:

```text
192.168.0.X:8080
10.X.X.X
172.16-31.X.X
```

The second case is important because it shows that the vulnerable backend can be used as a bridge into the internal network.

### 7. Use Burp Repeater First

Start with Burp Repeater to understand:

- what changes,
- what is blocked,
- what is fetched,
- which response differences matter.

Repeater is better for understanding.

### 8. Use Intruder When There Is a Range

Use Burp Intruder in legal labs when you need to test many values, such as:

- internal IP range,
- port range,
- path list,
- host list.

Example internal range pattern:

```text
http://192.168.0.§1§:8080/admin
```

Payload type:

```text
Numbers: 1 to 255
```

Compare:

- status code,
- response length,
- page title,
- body content,
- keywords such as `Admin`, `Users`, `Delete`.

Useful rule:

> SSRF + IP range = Intruder.

### 9. Map Any Filters Before Bypassing Them

If the application blocks a request, do not jump straight to random payloads.

First determine what is blocked:

- `localhost`?
- `127.0.0.1`?
- private IP ranges?
- `/admin`?
- specific protocols?
- specific ports?
- redirects?
- encoded values?

Example:

```text
localhost -> blocked
127.0.0.1 -> blocked
127.1 -> allowed
/admin -> blocked
/%2561dmin -> allowed
```

This shows a weak blacklist rather than robust destination validation.

### 10. Confirm the Security Impact

A useful SSRF finding should explain impact, not just payload.

Impact questions:

- Could the backend reach localhost?
- Could it access an internal admin panel?
- Could it reach another backend system?
- Could it trigger a state-changing action?
- Could it enumerate internal hosts or ports?
- Could it reach cloud metadata?
- Is the response visible or blind?

### 11. Think Like a Developer

After confirming the issue, ask:

- Why was user input allowed to define a backend request?
- Why did the backend trust localhost/internal services?
- Why was the internal admin interface not properly authenticated?
- Was the protection based on blacklist string matching?
- Was the final resolved destination validated?
- Were redirects handled safely?

## Practical Response Comparison

When comparing SSRF responses, check:

```text
Status code
Response length
Content-Type
Page title
HTML content
Error message
Redirect location
Response time
```

A single different response may identify the internal target.

## Common Mistake

A direct request to an internal admin endpoint may fail:

```http
GET /admin/delete?username=carlos
```

This can return `401 Unauthorized` because the request comes from the user/browser.

The SSRF attack must go through the vulnerable backend-controlled request parameter:

```http
POST /product/stock

stockApi=http://localhost/admin/delete?username=carlos
```

The difference is the source of the request.

## Safe Scope Reminder

Only perform SSRF testing in:

- legal labs,
- owned local environments,
- explicitly authorised test environments,
- bug bounty scopes where SSRF testing is allowed.

Internal IP scanning, port scanning, and metadata endpoint testing can be high-risk outside authorised labs.

## Key Takeaway

SSRF testing is not about memorising one payload.

It is about understanding:

```text
user-controlled destination -> backend request -> trust boundary crossed -> internal impact
```
