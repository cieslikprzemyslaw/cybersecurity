# PortSwigger Lab 02 - Web Shell Upload via Content-Type Restriction Bypass

## What I Tested

I tested whether the upload validation trusted the file part Content-Type header.

## What I Found

The application rejected the PHP file when it was sent as:

```http
Content-Type: application/octet-stream
```

The response said:

```text
Only image/jpeg and image/png are allowed
```

The same PHP file was accepted when the file part Content-Type was changed to:

```http
Content-Type: image/png
```

The filename remained:

```text
shell.php
```

The server later executed the uploaded PHP file.

## Root Cause

The application trusted a user-controlled multipart header as the main validation mechanism.

## Impact

An attacker could upload a PHP web shell disguised as an image and execute commands on the server.

## Regression Test Idea

Uploading `shell.php` with `Content-Type: image/png` should not result in an executable file being stored or accessible.

## Main Takeaway

`Content-Type` in a multipart upload request is a hint, not proof.
