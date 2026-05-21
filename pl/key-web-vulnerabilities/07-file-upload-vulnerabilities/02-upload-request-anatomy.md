# Anatomia requestu uploadu

## TL;DR

Większość uploadów używa `multipart/form-data`. Każdy plik jest osobną częścią body requestu i ma własne nagłówki.

## Przykład

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

## Content-Type części pliku

Główny `Content-Type` requestu to `multipart/form-data`.

`Content-Type` części pliku, np. `image/png`, dotyczy konkretnego pliku i można go zmienić w Burp.

## Co sprawdzać w Burp

- endpoint
- nazwa pola formularza
- nazwa pliku
- `Content-Type` części pliku
- zawartość pliku
- CSRF token
- odpowiedź
- final URL
- `Content-Type` odpowiedzi po wejściu w plik

## Główna lekcja

W testowaniu uploadu body requestu jest często ważniejsze niż UI.
