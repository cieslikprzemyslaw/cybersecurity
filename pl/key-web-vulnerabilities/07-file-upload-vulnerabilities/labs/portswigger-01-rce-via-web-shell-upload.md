# PortSwigger Lab 01 - Remote Code Execution via Web Shell Upload

## What I Tested

I tested whether the avatar upload feature allowed a PHP file to be uploaded and executed by the server.

## What I Found

The application accepted a PHP file and stored it in a web-accessible avatar directory.

Example:

```text
/files/avatars/shell.php?cmd=cat+/home/carlos/secret
```

## Root Cause

The application allowed executable server-side files to be uploaded and served from a location where the web server executed them.

## Impact

An attacker could execute commands on the server and read sensitive files.

## Remediation

- reject executable extensions
- store uploads outside executable web roots
- disable script execution in upload directories
- generate safe server-side filenames
- validate file type server-side

## Developer Takeaway

Never let user-uploaded files be executed as code.

## Main Takeaway

The dangerous part was not only the upload. The dangerous part was that the uploaded PHP file was executable.
