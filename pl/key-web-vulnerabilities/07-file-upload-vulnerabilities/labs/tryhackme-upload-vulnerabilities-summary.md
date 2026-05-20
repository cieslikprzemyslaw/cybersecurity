# TryHackMe Upload Vulnerabilities - Learning Summary

## Scope

This room introduced common file upload risks and filtering techniques.

VM nie działała stabilnie podczas nauki, więc room został użyty głównie jako materiał do czytania, a praktyka została przeniesiona do PortSwigger labs.

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

Upload accepted nie wystarczy. Check final filename, storage location, response Content-Type and whether the file is executed or served statically.
