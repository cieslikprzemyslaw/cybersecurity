# File Upload Remediation Cheatsheet

## TL;DR

Do not trust uploaded files. Validate server-side, store safely, rename files, and prevent code execution in upload directories.

## Key Controls

- Server-side validation
- Strict allowlists
- Server-generated filenames
- Safe storage outside executable web root where possible
- Disabled code execution in upload directories
- Safe response headers
- Size and count limits
- Malware/content scanning where appropriate
- Regression tests

## Do Not Trust User-Controlled Filenames

Risky:

```text
/uploads/shell.php
```

Safer:

```text
/uploads/8f4a9c2e-1d5b-4b8a.bin
```

## Serve Uploaded Files Safely

Use safe headers where appropriate:

```http
Content-Disposition: attachment
X-Content-Type-Options: nosniff
Content-Type: application/octet-stream
```

## Regression Test Ideas

The application should reject or safely handle:

```text
shell.php
shell.phtml
shell.phar
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
shell.PHP
SVG with script
HTML upload
oversized files
path traversal filenames
```

## Main Takeaway

The strongest fix is not a single filter. It is a safe upload design: validate, rename, store safely, and prevent execution.
