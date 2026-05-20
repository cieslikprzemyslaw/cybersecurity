# XSS Cheat Sheet — Practical Review Workflow

> This cheat sheet is for legal labs, local environments, and authorised testing only.

---

## Core Question

```text
Can attacker-controlled data reach a browser execution context without safe encoding or sanitization?
```

---

## 1. Find Input Points

Look for places where the user can submit or control data:

- search boxes,
- URL query parameters,
- URL path values,
- comments,
- profile fields,
- support tickets,
- chat messages,
- CMS/rich text fields,
- file names,
- uploaded SVG/HTML-like content,
- headers such as Referer or User-Agent in some edge cases,
- DOM sources such as `location.search` or `location.hash`.

---

## 2. Use a Unique Test String First

Before using any payload, start with a simple marker:

```text
xsstest123
```

Then check:

- Does it appear in the HTTP response?
- Does it appear in the DOM after JavaScript runs?
- Is it stored and displayed later?
- Where exactly does it appear?

---

## 3. Identify the Context

The context decides the test strategy.

| Context | Example | What to check |
|---|---|---|
| HTML body | `<p>USER_INPUT</p>` | Is output HTML-encoded? |
| HTML attribute | `<input value="USER_INPUT">` | Can quotes break out of the attribute? |
| JavaScript string | `var x = 'USER_INPUT'` | Is the string safely escaped? |
| URL | `<a href="USER_INPUT">` | Are dangerous schemes blocked? |
| DOM sink | `innerHTML = value` | Can source reach unsafe sink? |

---

## 4. Reflected XSS Workflow

1. Send a request with a unique marker.
2. Check whether the marker appears in the immediate response.
3. Identify the context.
4. Test whether the browser interprets the value as code.
5. Confirm whether the issue is exploitable only in a legal lab/scope.
6. Document root cause and remediation.

Key question:

```text
Does request input return immediately in the response without safe output encoding?
```

---

## 5. Stored XSS Workflow

1. Submit a unique marker into a stored field.
2. Revisit the page where the data is displayed.
3. Check whether the value is rendered to other users or privileged users.
4. Identify the context.
5. Test safely in the lab whether code execution is possible.
6. Document who would be affected.

Key question:

```text
Is untrusted data saved and later rendered in an unsafe way?
```

---

## 6. DOM XSS Workflow

Look for source-to-sink flows.

Common sources:

```js
location.search
location.hash
location.href
document.referrer
localStorage
sessionStorage
postMessage
```

Common sinks:

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

Workflow:

1. Add a marker to a URL parameter or hash.
2. Use DevTools to search the DOM for the marker.
3. Review JavaScript that handles the value.
4. Identify source and sink.
5. Replace unsafe rendering with safe APIs where possible.

Key question:

```text
Does browser-side JavaScript move attacker-controlled data into an unsafe sink?
```

---

## 7. Attribute Context Reminder

If input appears here:

```html
<input value="USER_INPUT">
```

and angle brackets are encoded, a tag-based payload may not work.

The lesson from the attribute-context lab:

```text
Break out of the existing attribute, then add a valid event handler.
```

Do not memorize only one payload. Understand what the browser will parse after the server renders the final HTML.

---

## 8. Framework and Rendering Pattern Review

Use the dedicated [Framework Risky Patterns](./framework-risky-patterns/README.md) notes for stack-specific review.

Quick routing:

| Area | Use |
|---|---|
| React, Angular, Vue, Svelte | Framework-specific raw HTML and URL review |
| Plain JavaScript / DOM APIs | Source-to-sink checks for unsafe DOM APIs |
| PHP / server-side templates | Context-aware output escaping checks |
| Markdown, CMS, rich text | Sanitization and allowlist review |

---

## 9. Remediation Quick Reference

| Problem | Better approach |
|---|---|
| Plain text rendered with `innerHTML` | Use `textContent` or safe JSX rendering |
| User input in HTML body | HTML output encode |
| User input in HTML attribute | Attribute encode and quote safely |
| User input in JavaScript string | JavaScript string encoding / avoid inline JS |
| User-provided HTML required | Sanitize using strict allowlist |
| CMS/rich text rendering | Sanitize and restrict allowed tags/attributes |
| XSS impact reduction | CSP as defence-in-depth |

---

## 10. Regression Test Ideas

Add tests that verify:

- payload-like input is displayed as text,
- comments cannot execute JavaScript,
- search terms are encoded in HTML responses,
- rich text sanitizer removes scripts and event handlers,
- `dangerouslySetInnerHTML` is not used with untrusted data,
- DOM sources do not reach unsafe sinks,
- uploaded SVG/HTML is not rendered unsafely.

---

## Main Takeaway

```text
Do not ask only “does alert run?”
Ask “what is the context, and is untrusted data being interpreted as code?”
```
