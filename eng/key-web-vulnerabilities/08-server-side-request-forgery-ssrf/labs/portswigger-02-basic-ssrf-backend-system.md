# PortSwigger Lab 02 - Basic SSRF Against Another Backend System

## Lab

PortSwigger Web Security Academy:

```text
Basic SSRF against another back-end system
```

## What I Tested

I tested the same type of stock check functionality, but this time the stock API pointed to an internal private IP address:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://192.168.0.1:8080/product/stock/check?productId=1&storeId=1
```

The normal response returned a stock number.

This showed that the application backend could communicate with an internal backend service on the `192.168.0.X` network.

## What I Found

The lab required finding an admin interface somewhere in the internal range:

```text
192.168.0.X:8080
```

Instead of testing each IP manually, I used Burp Intruder to iterate over the last octet.

Conceptual pattern:

```text
http://192.168.0.§1§:8080/admin
```

Payload range:

```text
1 to 255
```

I compared responses using:

- status code,
- response length,
- HTML title,
- response body,
- presence of admin-related content.

One IP returned an admin panel containing user deletion links.

After identifying the internal admin host, I used the vulnerable `stockApi` parameter to make the backend request the internal delete endpoint for `carlos`.

## Important Learning Moment

I did not immediately think about Burp Intruder.

The useful mental shortcut was:

> SSRF + internal IP range = use Intruder in a legal lab.

This is different from the first lab because the target was not localhost. The vulnerable backend could reach another internal service in the private network.

## Why It Matters

This lab showed that SSRF is not limited to the vulnerable server itself.

A vulnerable backend can become a proxy into the internal network.

If internal services trust the network location of the application server, SSRF can be used to access sensitive internal functionality that is not exposed to the internet.

## Root Cause

The root cause was:

- user-controlled backend request destination,
- backend access to internal private IP range,
- no strict allowlist for stock API destinations,
- internal admin service reachable from the application backend,
- internal service relying on network-level isolation rather than strong authentication.

## Impact

Possible impact:

- discovery of internal hosts,
- access to internal admin panels,
- unauthorized actions on backend systems,
- internal network reconnaissance,
- exposure of services that are not internet-facing,
- bypass of network-based access assumptions.

## Internal Network Reconnaissance

SSRF can be used to infer internal systems by testing:

```text
Different IP addresses
Different ports
Different paths
Different protocols
```

Signals may include:

```text
Different status codes
Different response lengths
Timeouts
Errors
HTML pages
Redirects
Specific keywords
```

This is why SSRF can be more serious than a simple external request issue.

## AppSec Principle Violated

The application violated this trust boundary:

> A public user-controlled request should not allow the backend to interact with arbitrary internal network services.

## Remediation

Safer implementation should:

- avoid letting users control backend service URLs,
- use server-side mappings for stock services,
- allow only known stock API hosts,
- block private IP ranges unless explicitly required,
- restrict ports,
- validate DNS resolution and final IP,
- re-check after redirects,
- isolate internal admin systems from application workloads,
- require authentication and authorization on internal admin services.

## Regression Test Idea

Add tests to ensure `stockApi` rejects private network targets:

```text
http://192.168.0.1:8080/admin
http://192.168.0.127:8080/admin
http://10.0.0.1:8080/admin
http://172.16.0.1:8080/admin
```

Expected result:

```text
The backend must not be usable as a proxy to scan or access internal hosts.
```

## Developer Takeaway

SSRF can turn a backend into an internal network bridge.

The safe fix is not only to block `localhost`. The application must control and validate all backend request destinations, including private IP ranges and redirected final targets.
