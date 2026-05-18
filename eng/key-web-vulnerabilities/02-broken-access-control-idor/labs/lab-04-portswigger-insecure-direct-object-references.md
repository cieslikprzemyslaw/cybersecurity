# Lab 04 - PortSwigger: Insecure Direct Object References

> **Platform:** PortSwigger Web Security Academy  
> **Topic:** Access Control / IDOR  
> **Status:** Completed  
> **Type:** Lab notes, not a full walkthrough

---

## Practice Focus

This lab showed that IDOR is not always a simple `?id=123` parameter.

The insecure object reference was related to chat transcript files stored and retrieved through predictable file references.

---

## Vulnerability Pattern

The application exposed direct references to transcript files.

If a user can change the referenced file and access another user's transcript, the backend is not enforcing access control for that file.

---

## What I Tested

- Where transcript files were referenced.
- Whether file names or URLs were predictable.
- Whether changing the file reference returned another transcript.
- Whether the backend checked if the current user was allowed to access that transcript.

---

## Root Cause

The backend allowed direct access to transcript files without verifying ownership or permission.

---

## Impact

An attacker could access another user's private conversation transcript.

In real applications, similar issues may expose:

- support tickets,
- chat logs,
- invoices,
- uploaded files,
- internal documents,
- private user data.

---

## Remediation

The backend should:

- avoid exposing predictable direct file references,
- map file access through an authorization layer,
- verify that the current user owns or is allowed to access the file,
- store files outside public web-accessible paths where appropriate,
- return `403` or `404` when access is not allowed.

---

## Main Takeaway

```text
IDOR can happen through files and URLs, not only through numeric IDs.
```
