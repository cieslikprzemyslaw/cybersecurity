# Where to Test Path Traversal / File Inclusion Payloads

## Purpose

Use this checklist when looking for places where user input may influence file access, file downloads, templates, images, language files or includes.

The main practical question is:

```txt
Where can user input influence which file the backend reads, loads or includes?
```

## High-signal parameters

| Parameter | Why it matters | Example |
|---|---|---|
| `file` | Often selects a file | `?file=report.pdf` |
| `filename` | Often used for images/downloads | `?filename=avatar.jpg` |
| `path` | May directly control filesystem path | `?path=/files/doc.pdf` |
| `page` | Often used in PHP include/navigation | `?page=about.php` |
| `lang` | May load language files | `?lang=en` |
| `template` | May load server-side templates | `?template=home` |
| `view` | May select a view/template | `?view=profile` |
| `document` | May return PDFs/docs | `?document=invoice.pdf` |
| `download` | May trigger file download | `?download=contract.pdf` |
| `image` / `img` | May fetch images from disk | `?image=1.jpg` |
| `avatar` | May fetch user-uploaded images | `?avatar=user.png` |

## Input locations to check

Payloads do not always go in the URL query string.

| Location | Example | Notes |
|---|---|---|
| Query string | `GET /image?filename=1.jpg` | Most common in labs |
| POST body | `file=report.pdf` | Common in forms/download actions |
| Cookies | `Cookie: file=admin` | Possible when apps use PHP `$_REQUEST` or custom logic |
| Headers | `X-File: report.pdf` | Less common, but possible in custom APIs |
| JSON body | `{"file":"report.pdf"}` | Common in modern APIs |
| Hidden fields | `<input type="hidden" name="file">` | Often missed in forms |
| Route params | `/download/report.pdf` | File path may be embedded in the route |

## Is a search box a good place to test?

Usually, no.

A normal search input such as:

```http
GET /search?q=test
```

usually sends the value to a search function or database query, not to filesystem access.

Search becomes interesting only if the value appears to select a file, template, report, export, page, document or downloadable resource.

## Request / response signals

Look for endpoints that load or return:

- Images.
- Avatars.
- Downloads.
- PDF/document previews.
- Attachments.
- Language files.
- Templates/views.
- Reports/exports.

Interesting responses include:

- `"No such file"`.
- `"File not found"`.
- PHP include warnings.
- Full filesystem paths in errors.
- `Content-Type` says image, but the body contains text.
- Different response length after a payload.
- `/etc/passwd`-like output.

## Basic testing flow

1. Find a request that loads a file.

```http
GET /image?filename=4.jpg
```

2. Change the value to a random string.

```http
GET /image?filename=test
```

3. Observe the response.

Look for errors such as `"No such file"` or response length changes.

4. Test basic traversal.

```http
GET /image?filename=../../../etc/passwd
```

5. If blocked, test alternative methods.

- Absolute path.
- Nested traversal.
- URL encoding.
- Double URL encoding.
- Required base path bypass.

6. Confirm whether the response contains file content.

7. Note which validation or file mapping failed.

## Key reminder

Do not test random inputs everywhere. First identify where user input appears to influence server-side file access.
