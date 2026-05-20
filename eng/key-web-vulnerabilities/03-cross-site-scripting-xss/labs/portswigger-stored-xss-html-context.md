# Lab Summary: PortSwigger — Stored XSS into HTML Context with Nothing Encoded

> **Platform:** PortSwigger Web Security Academy  
> **Topic:** Stored XSS  
> **Status:** Completed  
> **Note type:** Learning summary, not a full walkthrough

---

## What I Tested

I tested a feature where user-submitted content was saved by the application and later displayed on a page.

The important question was:

```text
Can data submitted by one user be stored and later rendered unsafely for other users?
```

---

## What I Found

The application stored untrusted input and later rendered it in an HTML context without safe output encoding or sanitization.

---

## Root Cause

The application trusted stored user-generated content and rendered it as active HTML.

```text
stored input → later response → browser execution
```

---

## Impact

Stored XSS can be more dangerous than reflected XSS because the victim may not need to click a crafted link. They only need to view the affected page.

Possible impact:

- attack users who read comments or messages,
- attack administrators viewing submitted content,
- perform actions as the victim,
- modify what users see in the application.

---

## Developer Remediation

- Encode output when rendering stored user content.
- Sanitize only if the application must support limited HTML/rich text.
- Use strict allowlists for allowed tags and attributes.
- Avoid rendering comments, profile fields, or support messages as raw HTML.
- Review admin dashboards carefully because stored XSS often affects privileged users.

---

## Regression Test Idea

Create content containing HTML-like input and verify it is displayed as text, not executed, for both normal users and admin users.

---

## Main Takeaway

```text
Stored XSS is dangerous because the payload becomes part of the application data and can affect future viewers.
```
