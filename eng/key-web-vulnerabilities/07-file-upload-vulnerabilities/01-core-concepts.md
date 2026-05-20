# File Upload Vulnerabilities - Core Concepts

## TL;DR

File upload vulnerabilities happen when an application accepts files from users but validates, stores, serves, or processes them unsafely.

The key question is not only:

> Can I upload a file?

The better AppSec question is:

> What does the application do with that file after upload?

## Why File Uploads Are Risky

An uploaded file is user-controlled input. The user may control:

- filename
- extension
- file part Content-Type
- file content
- file size
- metadata
- encoded or obfuscated filename characters

## Main Impact Areas

### Remote Code Execution

The most critical case is when an attacker uploads server-side code, such as PHP, and the server later executes it.

### Stored XSS

Uploaded files such as HTML or SVG may be rendered by the browser and execute JavaScript, especially if served from the same origin as the main application.

### File Overwrite

If the server saves files using user-controlled filenames, an attacker may try to overwrite existing files.

### Path Traversal Through Filename

If the filename is used unsafely during storage, path traversal may be possible.

### Information Disclosure

Upload features may reveal filesystem paths, upload locations, backend errors, predictable file URLs, or other users' files.

## Key Mental Model

A successful upload does not automatically mean code execution.

Always check:

- final stored filename
- final URL
- response Content-Type
- whether the file is served as static content
- whether the server executes the file
- whether access control is enforced

## Main Takeaway

File upload becomes dangerous when user-controlled files are accepted and then stored, served, parsed, or executed in an unsafe way.
