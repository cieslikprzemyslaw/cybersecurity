# SSRF Bypasses and Filtering Cheatsheet

## Purpose

Use this note to understand why simple SSRF filters often fail.

This is not a payload dump. It is a collection of patterns observed in legal labs and useful concepts for developer-focused remediation.

## Why Blacklists Are Weak

A blacklist tries to block known-bad values such as:

```text
localhost
127.0.0.1
/admin
```

This is fragile because attackers may use equivalent representations that reach the same destination but do not match the blocked string.

A safer design is to use strict allowlists, validate the final resolved destination, and restrict network access.

## Pattern 1: Blocking `localhost`

Blocked input:

```text
http://localhost/admin
```

Possible issue:

```text
The filter blocks the literal word "localhost".
```

But this does not necessarily block other loopback representations.

## Pattern 2: Blocking `127.0.0.1`

Blocked input:

```text
http://127.0.0.1/admin
```

Possible bypass pattern in labs:

```text
http://127.1/
```

Why it matters:

```text
127.1 can still resolve to a loopback address, but weak filters may only block the exact string 127.0.0.1.
```

Other equivalent IP representations may also exist depending on parser and environment.

## Pattern 3: Blocking Sensitive Paths

Blocked input:

```text
/admin
```

Possible issue:

```text
The filter searches for the literal string "admin".
```

If the backend decodes the URL later, encoded characters may still reach the intended path.

Example concept:

```text
/admin
/%61dmin
/%2561dmin
```

## Pattern 4: Double Encoding

Double encoding can bypass filters when validation and request processing decode input differently.

Example:

```text
%2561
```

Decode step 1:

```text
%61
```

Decode step 2:

```text
a
```

So:

```text
/%2561dmin
```

may eventually become:

```text
/admin
```

A weak blacklist may not detect the final path if it validates before all decoding/normalisation is complete.

## Pattern 5: URL Parser Confusion

Different parsers may interpret the same URL differently.

Examples to understand conceptually:

```text
https://expected-host@evil-host
https://evil-host#expected-host
https://expected-host.evil-host
```

Risk:

```text
Validation code may think the URL points to an allowed host, while the HTTP client requests a different final host.
```

## Pattern 6: Redirect-Based Bypass

If the backend follows redirects, an allowed URL may redirect to a blocked internal URL.

Flow:

```text
Backend validates allowed URL
Allowed URL returns redirect
Backend follows redirect to internal target
```

Risk:

```text
The application validates the first URL but not the final destination after redirects.
```

## Pattern 7: Open Redirect + SSRF

If an allowed application endpoint contains an open redirect, it may be chained with SSRF.

Conceptual flow:

```text
stockApi=https://allowed-host/redirect?path=http://internal-target/admin
```

The URL may pass the allowlist because it starts with or belongs to the allowed host, but the backend eventually follows the redirect to the internal target.

## Pattern 8: DNS-Based Issues

Potential DNS-related SSRF risks include:

- attacker-controlled domains resolving to internal IPs,
- DNS rebinding,
- validation before DNS resolution,
- different DNS resolution between validation and request time.

Developer note:

```text
Do not only validate the hostname string. Validate the resolved IP and final destination at request time.
```

## What To Compare During Filter Testing

When mapping SSRF filters, compare:

```text
localhost
127.0.0.1
127.1
private IP ranges
allowed external host
blocked path
encoded path
double-encoded path
different protocols
different ports
redirect behaviour
```

Record:

```text
Status code
Response body
Response length
Error message
Redirect location
```

## Developer Lesson

A filter that says:

```text
block if input contains "localhost" or "admin"
```

is not a robust SSRF defence.

A safer pattern is:

```text
parse URL safely
resolve hostname
validate scheme
validate port
validate final IP
block private/loopback/link-local/metadata ranges
follow redirects only if each hop is validated
allow only specific expected destinations
enforce network-level egress controls
```

## Key Takeaway

Blacklist-based SSRF protection fails because it usually validates strings, not the final network destination.

The important security question is:

> Where does the backend actually send the request after parsing, decoding, DNS resolution, redirects, and normalisation?
