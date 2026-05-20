# Lab Summary: PortSwigger — Reflected XSS into HTML Context with Nothing Encoded

> **Platform:** PortSwigger Web Security Academy  
> **Topic:** Reflected XSS  
> **Status:** Completed  
> **Note type:** Learning summary, not a full walkthrough

---

## What I Tested

I tested a search feature where user input was reflected back into the page.

The important question was:

```text
Does my input appear in the immediate HTML response?
```

---

## What I Found

The application reflected the search input directly into the HTML response without safely encoding it.

This created a reflected XSS issue because the browser could interpret attacker-controlled input as active content.

---

## Root Cause

The application rendered user-controlled request data into an HTML context without context-aware output encoding.

```text
request parameter → HTML response → browser interprets as markup/code
```

---

## Impact

An attacker could craft a link that causes JavaScript to execute in the victim's browser when they open it.

Depending on the application and victim privileges, this could lead to:

- actions performed as the victim,
- reading data visible to the victim,
- phishing-style UI manipulation,
- attacking privileged users.

---

## Developer Remediation

- Encode output before rendering user input in HTML.
- Treat query parameters as untrusted.
- Use safe template rendering defaults.
- Avoid manually concatenating user input into HTML.
- Add tests proving search terms are displayed as text, not executed as code.

---

## Regression Test Idea

Submit payload-like input to the search field and assert that it appears as escaped text in the response.

---

## Main Takeaway

```text
Reflected XSS is about unsafe immediate reflection of request input into the response.
```
