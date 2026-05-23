# SSRF Core Concepts

## TL;DR

Server-Side Request Forgery (SSRF) happens when an attacker can influence where a server-side application sends an HTTP request.

The key point is:

> The request is made by the backend server, not directly by the user's browser.

This matters because the backend may be able to access internal services, localhost-only interfaces, private IP ranges, cloud metadata endpoints, or other systems that are not directly reachable from the public internet.

## Basic Mental Model

Normal browser request:

```text
User browser -> public web application
```

SSRF request flow:

```text
User controls input -> application backend -> backend makes request to attacker-chosen destination
```

In SSRF, the attacker is not directly connecting to the internal target. Instead, they are abusing the trusted network position of the application server.

## Why SSRF Is Server-Side

SSRF is server-side because the vulnerable application takes user-controlled input and uses it to make a backend request.

Example:

```http
POST /product/stock

stockApi=http://stock.example.internal/product/stock/check?productId=1&storeId=1
```

If the backend fetches the value of `stockApi`, then changing that value may change where the server sends its request.

The dangerous part is not only that the parameter contains a URL. The dangerous part is that the backend trusts and fetches that URL.

## Why `localhost` Means the Server

In SSRF, this does not mean the user's laptop:

```text
http://localhost
http://127.0.0.1
```

It means the localhost of the application server.

So if a backend request is changed to:

```text
http://localhost/admin
```

the application server may request its own internal admin interface.

This can bypass access controls that incorrectly trust requests coming from the local machine.

## External URL vs Internal URL

An external URL is publicly reachable:

```text
https://example.com/image.jpg
https://api.public-service.com/data
```

An internal URL may only be reachable from inside the application's infrastructure:

```text
http://localhost/admin
http://127.0.0.1/admin
http://10.0.0.5/internal-api
http://192.168.0.10:8080/admin
```

A user may not be able to reach the internal URL directly, but the backend might.

## Private and Internal IP Ranges

Common internal or special-purpose ranges to remember:

```text
127.0.0.0/8       loopback / localhost
10.0.0.0/8        private network
172.16.0.0/12     private network
192.168.0.0/16    private network
169.254.0.0/16    link-local addresses
```

Important cloud metadata endpoint:

```text
169.254.169.254
```

This endpoint is commonly associated with cloud instance metadata services. If a vulnerable backend can reach it, SSRF may expose instance metadata, tokens, IAM role information, or other sensitive cloud configuration depending on the environment and protections in place.

## Regular SSRF vs Blind SSRF

### Regular / Non-Blind SSRF

Regular SSRF happens when the backend response is returned in the application's frontend response.

Example:

```text
stockApi=http://localhost/admin
```

The application returns the HTML of the internal admin page.

This is easier to detect because the attacker can directly see the result.

### Blind SSRF

Blind SSRF happens when the backend makes the request, but the response is not returned to the user.

Detection may require indirect evidence such as:

- a callback to a controlled server,
- Burp Collaborator interaction,
- timing differences,
- different error messages,
- different status codes.

Blind SSRF is harder to prove, but it can still be serious.

## Why SSRF Can Be Dangerous

SSRF can allow an attacker to:

- access localhost-only admin panels,
- access internal APIs,
- reach backend systems on private IP addresses,
- perform internal network reconnaissance,
- interact with cloud metadata endpoints,
- trigger unauthorized state-changing actions,
- leak sensitive data from trusted internal services.

The real impact depends on what the vulnerable backend can reach.

## SSRF Compared With Previous Topics

### Path Traversal

```text
Attacker controls a file path used by the server.
```

### File Upload

```text
Attacker controls a file that is stored or processed by the server.
```

### SSRF

```text
Attacker controls a destination that the server requests.
```

The payload in SSRF is often a URL, host, path, or internal destination rather than a file path, uploaded file, or SQL fragment.

## Key Takeaway

SSRF abuses server-side trust.

The attacker does not need direct access to the internal system. They only need a way to make the trusted backend send a request to it.
