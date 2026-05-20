# Upload Request Anatomy

## TL;DR

Most file uploads are sent using `multipart/form-data`. Each file is sent as one part of the request body, with its own headers.

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

```http
Content-Disposition: form-data; name="avatar"; filename="php-marker.php"
```

- `name="avatar"` is the form field name.
- `filename="php-marker.php"` is user-controlled.

The filename should never be trusted directly.

## File Part Content-Type

Top-level request Content-Type:

```http
Content-Type: multipart/form-data; boundary=...
```

File part Content-Type:

```http
Content-Type: image/png
```

The file part Content-Type is controlled by the client and can be changed in Burp.

## What to Inspect in Burp

- upload endpoint
- file field name
- filename
- file part Content-Type
- file content
- CSRF token
- response message
- final uploaded file URL
- response Content-Type when accessing the uploaded file

## Main Takeaway

In file upload testing, the request body is often more important than the UI.
