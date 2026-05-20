# PortSwigger Lab 03 - Web Shell Upload via Obfuscated File Extension

## What I Tested

I tested how the application handled filename and extension validation.

## What I Found

```text
shell.php                  -> rejected
shell.php + image/png      -> rejected
shell.php.jpg              -> accepted, but served as image
shell.jpg.php              -> rejected
shell.php%00.jpg           -> accepted and stored as shell.php
shell.php?cmd=...          -> executed after successful storage as .php
```

## Why `shell.php.jpg` Was Not Enough

The upload was accepted, but the final file was served as an image:

```http
Content-Type: image/jpeg
```

The PHP code appeared in the response body as text, which showed that the server did not execute it.

## Why `shell.php%00.jpg` Worked

The validation accepted the filename because it appeared to end with an allowed image extension.

However, the application stored the file as:

```text
shell.php
```

The uploaded file was then executed as PHP by the server.

## Root Cause

Weak filename/extension validation and unsafe handling of encoded/null-byte characters in the filename.

## Regression Test Idea

The application should reject or safely handle:

```text
shell.php
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
shell.php%00.png
shell.PHP
shell.pHp
shell%2ephp.jpg
```

## Main Takeaway

The key bug was the difference between what the validation saw and what the application finally stored.
