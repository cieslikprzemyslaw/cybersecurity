# PortSwigger 03 - Traversal Sequences Stripped Non-Recursively

## What I tested

I tested an image-loading endpoint:

```http
GET /image?filename=5.jpg
```

Basic payloads such as `/etc/passwd` and `../../../../etc/passwd` did not work.

## What I found

Nested traversal worked:

```http
GET /image?filename=....//....//....//etc/passwd
```

## Why it matters

The application attempted to remove traversal sequences, but the filtering was weak and non-recursive.

## Root cause

The backend likely removed `../` once, but did not re-check the resulting path after transformation.

## Impact

An attacker could bypass weak filtering and read sensitive files.

## Note to self

When filtering removes a pattern once, nested payloads can create the dangerous pattern after filtering.
