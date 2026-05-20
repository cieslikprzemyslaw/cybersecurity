# Validation and Filter Bypass Cheatsheet

## TL;DR

Upload validation is often made of multiple weak checks. A single check is rarely enough.

## Client-Side Validation

Client-side validation runs in the browser. It can improve UX, but it is not a security boundary.

### Four Client-Side Bypass Ideas

1. Disable JavaScript.
2. Modify the incoming HTML/JavaScript response in Burp.
3. Modify the upload request in Burp.
4. Send the request directly to the upload endpoint with Burp, curl, Postman, or a script.

## Server-Side Validation

Server-side validation is tested by enumeration because you usually cannot see the code.

## Extension Validation Test Cases

```text
shell.php
shell.phtml
shell.phar
shell.php5
shell.jpg.php
shell.php.jpg
shell.PHP
```

## Blacklist vs Allowlist

A blacklist blocks known-bad extensions. It is fragile because the developer must predict every dangerous extension and variation.

An allowlist allows only expected extensions such as:

```text
.jpg
.jpeg
.png
.webp
```

## MIME Type / Content-Type Validation

A weak server may trust the file part Content-Type:

```http
Content-Type: image/png
```

This can be changed by the attacker.

Lab observation:

```text
application/octet-stream -> rejected
image/png -> accepted
```

## Magic Bytes / Magic Numbers

Magic bytes are the first bytes of a file that identify its format.

```text
PNG  -> 89 50 4E 47 0D 0A 1A 0A
JPEG -> FF D8 FF DB
```

Magic byte validation is stronger than checking only extension or Content-Type, but it is not complete protection.

## Obfuscated File Extension Bypass

Useful legal-lab test cases:

```text
shell.php
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
shell.php%00.png
shell.PHP
shell.pHp
shell.php%2ejpg
shell%2ephp.jpg
```

The goal is not to memorise payloads. The goal is to understand filename handling at different stages:

1. validation
2. storage
3. final URL
4. server-side execution

### PortSwigger Lab Observation

```text
shell.php         -> rejected
shell.php.jpg     -> accepted, but served as image
shell.jpg.php     -> rejected
shell.php%00.jpg  -> accepted and stored as shell.php
```

Key idea:

> The filter saw a JPG-like filename, but backend/storage handled the obfuscated filename differently and stored an executable `.php` file.

## Main Takeaway

Upload validation must be layered. Extension checks, Content-Type checks and magic bytes are useful signals, but safe storage and disabled execution in upload directories are critical.
