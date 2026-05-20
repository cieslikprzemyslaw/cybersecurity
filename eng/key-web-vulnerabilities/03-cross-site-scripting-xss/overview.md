# Cross-Site Scripting (XSS) — AppSec Learning Notes

> **Learning path:** Frontend Engineer → Application Security
> **Sources:** TryHackMe + PortSwigger Web Security Academy
> **Topic:** Reflected XSS, Stored XSS, DOM XSS, source, sink, context, output encoding, safe rendering
> **Status:** Learning notes, not a full lab walkthrough

---

## TL;DR

**XSS happens when untrusted data is rendered in a way that lets the browser treat it as code instead of text.**

The important question is not only:

> Can I run JavaScript?

The better AppSec question is:

> Where does user-controlled data enter the application, and where is it rendered?

```text
Untrusted input + unsafe browser context = possible XSS
```

---

## What is XSS?

Cross-Site Scripting is a client-side injection issue. It allows attacker-controlled JavaScript or HTML-like content to execute in another user's browser in the context of the vulnerable application.

Depending on the application and the victim's permissions, XSS may allow an attacker to:

- perform actions as the victim,
- read data visible to the victim,
- modify the page,
- capture sensitive input,
- abuse application functionality,
- attack admin users if the payload executes in an admin context.

The impact depends heavily on:

- where the payload runs,
- who views it,
- what the application allows that user to do,
- whether sensitive data or privileged actions are available.

---

## Main Types of XSS

### Reflected XSS

Reflected XSS happens when data from the current HTTP request is included in the immediate response in an unsafe way.

Typical pattern:

```text
GET /search?q=userInput
```

The application reflects `userInput` into the HTML response without safe output encoding.

Key idea:

```text
request input → immediate response → browser execution
```

This is often delivered through a crafted link.

---

### Stored XSS

Stored XSS happens when untrusted input is saved by the application and later rendered in responses shown to users.

Common locations:

- blog comments,
- support tickets,
- user profiles,
- product reviews,
- chat messages,
- admin dashboards,
- CMS content.

Key idea:

```text
input saved → later rendered → browser execution for future viewers
```

Stored XSS can be more dangerous than reflected XSS because the victim does not need to click a specially crafted link. They may only need to open a page where the payload was stored.

---

### DOM XSS

DOM XSS happens when client-side JavaScript takes attacker-controlled data and writes it into the page using an unsafe API.

Key idea:

```text
browser-controlled source → unsafe JavaScript sink → DOM execution
```

The payload may not appear in the original server response. It may only appear after JavaScript runs in the browser.

Example risky flow:

```js
const name = new URLSearchParams(window.location.search).get("name");
document.getElementById("welcome").innerHTML = name;
```

Here:

- source: `window.location.search`
- sink: `innerHTML`

If `name` is attacker-controlled, this may be DOM XSS.

---

## Source and Sink

### Source

A **source** is where attacker-controlled data comes from.

Common DOM XSS sources:

```js
location.href
location.search
location.hash
document.referrer
localStorage
sessionStorage
postMessage
```

### Sink

A **sink** is where data is used in a potentially unsafe way.

Common dangerous sinks:

```js
innerHTML
outerHTML
document.write()
insertAdjacentHTML()
eval()
new Function()
setTimeout("string")
setInterval("string")
```

A useful code review question:

```text
Can attacker-controlled data reach an unsafe sink?
```

---

## XSS Context Matters

The same payload does not work everywhere. XSS depends on the context where input is rendered.

Common contexts:

- HTML body,
- HTML attribute,
- JavaScript string,
- URL,
- CSS,
- DOM after JavaScript execution.

### HTML Body Context

Example:

```html
<p>You searched for: USER_INPUT</p>
```

If the application does not encode output, HTML-like payloads may be interpreted as markup.

### HTML Attribute Context

Example:

```html
<input value="USER_INPUT">
```

If `<` and `>` are encoded, creating a new tag may not work. The attacker may try to break out of the existing attribute and add another attribute/event handler.

Example lab lesson:

```text
" closes the existing value attribute
an event handler such as onfocus can execute JavaScript
```

The important lesson is not the exact payload. The important lesson is understanding the context and how the browser parses the final HTML.

---

## Sanitization vs Output Encoding

### Sanitization

Sanitization removes or neutralizes unsafe content.

Example goal:

```text
Remove unsafe tags, attributes, scripts, event handlers, or dangerous URLs from rich text.
```

Sanitization is often needed when the application intentionally allows some HTML, for example rich text or CMS content.

### Output Encoding

Output encoding converts special characters so the browser displays them as text instead of interpreting them as markup or code.

Example:

```html
<script>
```

becomes:

```html
&lt;script&gt;
```

The browser displays the text, but does not execute it.

Key point:

```text
Output encoding must be context-aware.
```

HTML body, HTML attributes, JavaScript strings, URLs, and CSS require different handling.

---

## Framework and Rendering Risky Patterns

Modern frameworks usually make normal text rendering safer, but XSS can reappear when code bypasses those defaults.

Review carefully when code uses:

- raw HTML rendering,
- direct DOM manipulation,
- framework escape/sanitization bypass APIs,
- markdown, CMS, or rich text rendered as HTML,
- user-controlled URLs.

Stack-specific notes live in [Framework Risky Patterns](./framework-risky-patterns/README.md).

---

## What I Practised

| Platform | Lab / Topic | Main Lesson |
|---|---|---|
| TryHackMe | Intro to Cross-site Scripting | XSS types, basic payloads, reflected/stored/DOM idea |
| PortSwigger | Reflected XSS into HTML context with nothing encoded | User input reflected into HTML can execute if not encoded |
| PortSwigger | Stored XSS into HTML context with nothing encoded | Stored payloads can execute later for other users |
| PortSwigger | DOM XSS in `document.write` sink using `location.search` source | DOM XSS is about source-to-sink flow in browser JavaScript |
| PortSwigger | Reflected XSS into attribute with angle brackets HTML-encoded | Payload must match the rendering context; attribute context needs different thinking |

---

## Developer Remediation

Good practices:

- treat all user-controlled input as untrusted,
- encode output based on the rendering context,
- avoid unsafe DOM sinks such as `innerHTML` when text rendering is enough,
- use `textContent` instead of `innerHTML` for plain text,
- avoid `dangerouslySetInnerHTML` unless required,
- sanitize rich text/HTML with a proven sanitizer and strict allowlist,
- validate input on arrival, but do not rely on validation alone,
- use safe framework defaults correctly,
- add CSP as defence-in-depth, not as the only fix,
- review markdown/CMS/rich text rendering carefully,
- write regression tests for dangerous input being displayed as text.

Unsafe pattern:

```js
element.innerHTML = userInput;
```

Safer pattern for plain text:

```js
element.textContent = userInput;
```

Safer React pattern:

```jsx
<p>{userInput}</p>
```

---

## Review Checklist

When reviewing code or testing an application, ask:

- Where does this input come from?
- Can a user control it?
- Where is it rendered?
- Is it rendered as text, HTML, JavaScript, URL, or CSS?
- Is the output encoding correct for that context?
- Is any raw HTML rendering used?
- Is `innerHTML`, `document.write`, or `dangerouslySetInnerHTML` used?
- Is markdown/rich text/CMS content sanitized?
- Could the same data be rendered later for another user?
- Is there a regression test proving payload-like input is displayed safely?

---

## OWASP / AppSec Mapping

Relevant category:

- **OWASP Top 10: Injection / XSS-related client-side injection**

Related principles:

- never trust user input,
- context-aware output encoding,
- safe rendering APIs,
- defence in depth,
- secure defaults,
- avoid unsafe browser sinks.

---

## Main Takeaway

XSS is about context.

```text
The same input can be safe in one context and dangerous in another.
```

As a Frontend Engineer, the most important habit is to trace the data flow:

```text
source → transformation → rendering context → browser behaviour
```
