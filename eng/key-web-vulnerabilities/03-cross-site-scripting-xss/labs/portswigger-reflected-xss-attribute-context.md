# Lab Summary: PortSwigger — Reflected XSS into Attribute with Angle Brackets HTML-Encoded

> **Platform:** PortSwigger Web Security Academy  
> **Topic:** XSS context / HTML attribute context  
> **Status:** Completed  
> **Note type:** Learning summary, not a full walkthrough

---

## What I Tested

I tested a reflected XSS case where user input appeared inside an HTML attribute rather than directly between HTML tags.

The important question was:

```text
What context does my input appear in?
```

---

## What I Found

The input appeared in an attribute similar to:

```html
<input value="USER_INPUT">
```

Angle brackets were HTML-encoded, so simply trying to create a new tag was not the right approach.

The key lesson was understanding how to break out of the existing attribute context and add a valid event handler.

---

## Root Cause

The application placed user-controlled input into an HTML attribute without safely encoding the full attribute context.

---

## Why Context Mattered

A basic HTML-body payload may fail in an attribute context.

In this lab, the useful idea was:

```text
close the current attribute → add a real browser event handler → keep the HTML parseable
```

This reinforced that XSS is not about memorising one payload. It is about understanding how the browser parses the final HTML.

---

## Impact

An attacker could craft a request where reflected input modifies the generated HTML attribute and causes JavaScript execution in the victim's browser.

---

## Developer Remediation

- Use context-aware output encoding for HTML attributes.
- Quote attributes safely.
- Avoid writing user input directly into attributes unless required.
- Validate input based on expected format.
- Use safe framework/template defaults and avoid manual HTML construction.

---

## Regression Test Idea

Verify that quotes, angle brackets, and event-handler-like input are safely encoded when rendered inside attributes.

---

## Main Takeaway

```text
Payloads depend on context. HTML body context and HTML attribute context require different thinking.
```
