# Anatomia upload requestu

## TL;DR

Większość uploadów używa `multipart/form-data`. Każdy plik jest osobną częścią request body i ma własne nagłówki.

## Example

```http
POST /my-account/avatar HTTP/2
Host: example.com
Content-Type: multipart/form-data; boundary=----boundary

------boundary
Content-Disposition: form-data; name="avatar"; filename="php-marker.php"
Content-Type: image/png

<?php echo "PHP_EXECUTION_TEST_OK"; ?>

------boundary--
```

## Content-Disposition

`filename` jest kontrolowany przez użytkownika i nie powinien być zaufany.

## File Part Content-Type

Top-level request Content-Type to `multipart/form-data`.

File part Content-Type, np. `image/png`, dotyczy konkretnego pliku i można go zmienić w Burp.

## Co sprawdzać w Burp

- endpoint
- field name
- filename
- file part Content-Type
- content pliku
- CSRF token
- response
- final URL
- response Content-Type po wejściu w plik

## Main takeaway

W file upload testing request body jest często ważniejsze niż UI.
