# Cross-Site Scripting (XSS) — AppSec Learning Notes

> **Learning path:** Frontend Engineer → Application Security  
> **Sources:** TryHackMe + PortSwigger Web Security Academy  
> **Topic:** Reflected XSS, Stored XSS, DOM XSS, source, sink, output encoding, safe rendering  
> **Status:** Learning notes, not a full lab walkthrough

---

## TL;DR

**XSS is not only about injecting `<script>`.**  
The real issue is that untrusted data is rendered in a place where the browser can treat it as code.

```text
Untrusted data + unsafe rendering context = potential XSS
```

For me as a frontend developer, the key question is:

> Is this value being rendered as safe text, or can the browser interpret it as HTML/JavaScript?

---

## Labs / Topics Covered

| Platform | Lab / Topic | What I Practiced |
|---|---|---|
| TryHackMe | Intro to Cross-site Scripting | Reflected, stored, and DOM XSS basics |
| PortSwigger | Reflected XSS into HTML context with nothing encoded | Finding reflected input in HTML response |
| PortSwigger | Stored XSS into HTML context with nothing encoded | Understanding stored payload execution |
| Review / notes | DOM XSS concepts | Source, sink, DOM flow, safe rendering APIs |

---

## What is XSS?

Cross-Site Scripting happens when an application places untrusted data into a page in a way that allows the browser to treat it as executable code.

The problem is not only that the user typed something dangerous. Users can always control input.

The real problem is that the application failed to handle that data safely before rendering it.

---

## Reflected XSS

Reflected XSS happens when user input from the request is immediately included in the response.

Example pattern:

```http
GET /search?q=test
```

If the value of `q` is included in the HTML response without proper encoding, it may become exploitable depending on the context.

Useful questions during testing:

- Which parameter do I control?
- Does my value appear in the response?
- Is it encoded or returned raw?
- What context is it reflected into?
- Would a victim need to click a crafted link?

---

## Stored XSS

Stored XSS happens when the malicious input is saved by the application and later displayed to users.

Common places:

- comments,
- user profiles,
- support tickets,
- chat messages,
- CMS content,
- product reviews,
- admin panels.

This can be more serious than reflected XSS because the payload can execute for every user who views the affected page.

A useful real-world question is:

> Who will view this stored content, and what privileges do they have?

Stored XSS in an admin panel or support dashboard can have a much higher impact than XSS visible only to the attacker.

---

## DOM XSS

DOM XSS happens when JavaScript running in the browser takes attacker-controlled data and writes it into the page using an unsafe API.

Simple pattern:

```text
attacker-controlled source → unsafe sink → browser execution
```

Example:

```js
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('welcome').innerHTML = name;
```

In this example:

- source: `window.location.search`
- sink: `innerHTML`

A safer version, when displaying text, would be:

```js
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('welcome').textContent = name;
```

---

## Source and Sink

### Source

A source is where attacker-controlled data comes from.

Examples:

- `location.search`
- `location.hash`
- `location.href`
- `document.referrer`
- `localStorage`
- `sessionStorage`
- `postMessage`

### Sink

A sink is where the data is used in a potentially unsafe way.

Examples:

- `innerHTML`
- `outerHTML`
- `document.write()`
- `insertAdjacentHTML()`
- `eval()`
- `setTimeout(string)`
- React `dangerouslySetInnerHTML`

---

## Why Context Matters

The same input can be safe in one context and dangerous in another.

Data may be inserted into:

- HTML body,
- HTML attribute,
- JavaScript string,
- URL,
- CSS,
- DOM API.

That is why XSS testing is not only about checking whether a value appears on the page.

The better question is:

> Where does the value appear, and how will the browser interpret it?

---

## Frontend Developer Takeaway

React helps because it escapes values rendered in JSX by default.

Usually safer:

```jsx
<p>{userInput}</p>
```

Riskier:

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

However, React does not automatically protect against every XSS issue.

XSS can still appear through:

- `dangerouslySetInnerHTML`,
- direct DOM manipulation,
- unsafe markdown or rich text rendering,
- CMS content,
- third-party components,
- unsafe URL handling,
- poorly sanitized HTML from APIs.

The frontend code review pattern I want to remember is:

```text
user-controlled data → unsafe rendering API
```

---

## How to Prevent XSS

Good practices:

- Use context-aware output encoding.
- Treat user input as text, not HTML.
- Use `textContent` instead of `innerHTML` when displaying plain text.
- Sanitize rich text/HTML if the application really needs to allow HTML.
- Avoid `eval()`, `document.write()`, inline event handlers, and unsafe DOM APIs.
- Avoid React `dangerouslySetInnerHTML` unless the HTML is trusted or properly sanitized.
- Use Content Security Policy as an additional layer, not as the only defense.
- Add regression tests for risky rendering paths.

---

## OWASP / AppSec Mapping

Relevant ideas:

- injection-style client-side vulnerability,
- input validation,
- output encoding,
- secure rendering,
- defense in depth,
- least privilege for session and token exposure.

Developer principles:

- never trust user input,
- encode output based on context,
- avoid unsafe browser APIs,
- sanitize only when HTML must be allowed,
- do not rely only on frontend validation.

---

## Review Checklist

When reviewing a frontend feature, I should ask:

- Does this data come from the user or another untrusted source?
- Is it rendered as text or HTML?
- Does it enter `innerHTML`, `outerHTML`, `document.write()`, `eval()`, or `dangerouslySetInnerHTML`?
- Is the output encoding appropriate for the context?
- If HTML is allowed, is it sanitized with a trusted sanitizer and an allowlist?
- Can this content be viewed by another user or an admin?
- Would CSP reduce impact if something was missed?
- Is there a regression test for this rendering path?

---

## Main Takeaway

The biggest lesson for me was that XSS is about unsafe data flow into the browser.

As a frontend developer, this is directly connected to everyday work: rendering API data, CMS content, markdown, form values, URL parameters, and rich text.

The browser will execute code if we accidentally give it code.

So the key habit is:

> Track the data flow from source to sink, then make sure untrusted data is rendered safely.
