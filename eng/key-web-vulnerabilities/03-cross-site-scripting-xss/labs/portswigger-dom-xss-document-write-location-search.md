# Lab Summary: PortSwigger — DOM XSS in document.write Sink Using location.search Source

> **Platform:** PortSwigger Web Security Academy  
> **Topic:** DOM XSS  
> **Status:** Completed  
> **Note type:** Learning summary, not a full walkthrough

---

## What I Tested

I tested a DOM-based XSS pattern where JavaScript running in the browser reads data from the URL and writes it into the page.

The important question was:

```text
Can attacker-controlled browser data reach an unsafe DOM sink?
```

---

## Source and Sink

Source:

```js
location.search
```

Sink:

```js
document.write()
```

---

## What I Found

The vulnerable flow was browser-side rather than server-side.

The server response alone did not fully explain the vulnerability. The issue happened when JavaScript processed URL-controlled data and wrote it into the DOM unsafely.

---

## Root Cause

Client-side JavaScript used attacker-controlled data from the URL and passed it into a dangerous sink.

```text
location.search → JavaScript processing → document.write() → DOM execution
```

---

## Impact

An attacker could craft a URL that causes JavaScript execution in the victim's browser when the vulnerable page processes the URL parameter.

---

## Developer Remediation

- Avoid `document.write()` for user-controlled data.
- Avoid unsafe DOM sinks such as `innerHTML`, `outerHTML`, and `insertAdjacentHTML` for untrusted input.
- Use `textContent` for plain text.
- Validate and encode data before rendering.
- Review source-to-sink flows during frontend code review.

---

## Regression Test Idea

Add a test or review rule that verifies URL parameters are not written into the DOM through unsafe sinks.

---

## Main Takeaway

```text
DOM XSS is about source-to-sink flow in browser-side JavaScript.
```
