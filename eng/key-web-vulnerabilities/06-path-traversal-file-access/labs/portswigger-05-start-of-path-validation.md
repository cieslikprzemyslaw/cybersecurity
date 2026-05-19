# PortSwigger 05 - Validation of Start of Path

## What I tested

The normal image request used a full expected path:

```http
GET /image?filename=/var/www/images/46.jpg
```

This suggested that the application expected the path to start with:

```txt
/var/www/images/
```

## What I found

The following payload worked:

```http
GET /image?filename=/var/www/images/../../../etc/passwd
```

## Why it matters

The input started with the expected base path, but the final resolved path escaped that directory.

## Root cause

The application likely checked the beginning of the string rather than the final canonical path.

## Impact

An attacker could satisfy the weak prefix check and still read sensitive files outside the image directory.

## Note to self

Prefix checks can look correct on the raw string while the resolved path points somewhere else.
