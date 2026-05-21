# Black-Box File Upload Testing Methodology

## TL;DR

Do not start with a payload. Start by understanding what the application accepts, blocks, stores and serves.

## Step 1: Identify the Technology Stack

Look for:

- `Server` header
- `X-Powered-By` header
- framework fingerprints
- Wappalyzer results
- error messages

## Step 2: Find Upload Points

Examples:

- avatar upload
- profile image
- media upload
- document attachment
- CSV import
- support ticket attachment

## Step 3: Inspect Client-Side Validation

Check:

- HTML `accept` attribute
- JavaScript extension checks
- JavaScript MIME checks
- file size checks

## Step 4: Upload a Harmless Valid File First

Upload a normal `.jpg` or `.png`.

Observe:

- response message
- final URL
- final filename
- whether the file is renamed
- whether it appears in the page

## Step 5: Find Where the File Is Stored or Served

Use:

- page source
- Burp HTTP history
- direct URL access
- content discovery
- Gobuster where appropriate

## Step 6: Compare Accepted and Rejected Files

```text
test.jpg          -> accepted
shell.php         -> rejected
shell.php.jpg     -> accepted but not executed
shell.jpg.php     -> rejected
shell.php%00.jpg  -> accepted and stored differently
```

## Step 7: Identify the Server-Side Filter

- Extension filter
- MIME / Content-Type filter
- Magic bytes filter
- File size filter
- Filename filter

## Step 8: Check How the File Is Served

Ask:

- Is the file returned as static content?
- Does the server execute it?
- What is the response Content-Type?
- Is the final filename what I expected?
- Is authentication required?

## Step 9: Separate Acceptance from Execution

An accepted upload is not automatically exploitable.

Example observations:

```text
shell.php  -> rejected
shell.php7 -> accepted, but served as static text
shell.l33t -> accepted and executed after server configuration changed
```

For every accepted dangerous-looking file, verify how the server handles it:

- rendered as an image
- downloaded as a file
- served as source text
- executed as server-side code

## Step 10: Check Server-Specific Behaviour

Use stack fingerprints to choose the next test branch.

Examples:

- Apache may make `.htaccess` relevant if local overrides are enabled.
- Nginx does not use `.htaccess`, so the same branch is usually irrelevant.
- PHP execution depends on how the web server and PHP handler are configured.
- Cloud object storage often serves uploaded files as static content only, but may introduce public access and stored XSS risks.

Apache blacklist-bypass pattern:

```text
1. Upload .htaccess with AddType application/x-httpd-php .custom
2. Upload PHP payload as shell.custom
3. Request /files/avatars/shell.custom?cmd=whoami
```

Key lesson:

```text
Upload accepted does not mean code execution.
Upload rejected does not mean every execution path is closed.
```

## Main Takeaway

Black-box file upload testing is enumeration-driven. Compare accepted and rejected files before choosing a bypass.
