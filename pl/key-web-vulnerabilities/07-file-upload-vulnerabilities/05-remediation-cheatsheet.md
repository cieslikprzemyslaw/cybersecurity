# File Upload Remediation Cheatsheet

## TL;DR

Nie ufaj uploadowanym plikom. Waliduj server-side, zapisuj bezpiecznie, zmieniaj nazwy plików i blokuj code execution w upload directories.

## Key Controls

- server-side validation
- strict allowlists
- server-generated filenames
- safe storage outside executable web root
- disabled code execution in upload directories
- safe response headers
- size limits
- regression tests

## Regression Test Ideas

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

## Main takeaway

Najmocniejsza poprawka to bezpieczny design uploadu: validate, rename, store safely, prevent execution.
