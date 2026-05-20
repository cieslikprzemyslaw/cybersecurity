# TryHackMe Upload Vulnerabilities - Learning Summary

## Scope

This room introduced common file upload risks and filtering techniques.

The VM did not work reliably during my study session, so I used the room mainly as reading material and moved hands-on practice to PortSwigger labs.

## Key Concepts

- file overwrite risk
- RCE through uploaded files
- client-side filtering
- server-side extension filtering
- MIME / Content-Type validation
- magic bytes
- black-box methodology

## Client-Side Filtering Bypasses

- disable JavaScript
- modify the incoming page response in Burp
- modify the upload request in Burp
- send the request directly to the upload endpoint

## Main Takeaway

Upload accepted is not enough. Check final filename, storage location, response Content-Type and whether the file is executed or served statically.
