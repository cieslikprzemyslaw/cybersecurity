# XSS Risky Patterns by Framework and Rendering Context

> **Learning path:** Frontend Engineer → Application Security
> **Topic:** Cross-Site Scripting, framework-specific risky patterns, secure rendering
> **Status:** Developer-focused review notes, not a payload collection

## Purpose

This folder collects common XSS-prone patterns across frontend frameworks, plain JavaScript, PHP, server-side templates, markdown, CMS and rich text rendering.

The goal is not to memorise payloads. The goal is to understand where XSS appears in real code and how to review it safely.

## Main idea

Most modern frameworks are safer by default when developers use normal text rendering.

XSS risk usually increases when developers bypass those protections and render untrusted data as raw HTML, JavaScript, URL, or DOM content.

```text
User-controlled data + unsafe rendering context = possible XSS
```

## Files

| File | Focus |
|---|---|
| `react.md` | React, JSX escaping, `dangerouslySetInnerHTML`, unsafe URL/rendering patterns |
| `angular.md` | Angular sanitization, `bypassSecurityTrust*`, raw DOM usage |
| `vue.md` | Vue interpolation, `v-html`, dynamic URLs |
| `svelte.md` | Svelte escaping, `{@html}` |
| `plain-javascript-dom.md` | `innerHTML`, `document.write`, `eval`, unsafe DOM sinks |
| `php.md` | PHP output escaping, `htmlspecialchars`, HTML/attribute/JS contexts |
| `server-side-templates.md` | Template engines such as Twig, Jinja, EJS, Handlebars, Razor, Thymeleaf |
| `markdown-cms-rich-text.md` | Markdown, CMS rich text, HTML sanitization, allowlists |

## Review questions

When reviewing code, ask:

- Is the data user-controlled?
- Is it rendered as text or as raw HTML?
- Is the framework protection being bypassed?
- Is the output context HTML body, HTML attribute, JavaScript, URL, CSS, or DOM?
- Is sanitization required?
- Is output encoding context-aware?
- Are `javascript:` URLs, event handlers, or unsafe tags possible?
- Is CSP used as defence-in-depth, not as the only protection?

## Developer takeaway

Frameworks help, but they do not remove the need for secure rendering decisions.

```text
Safe defaults protect normal rendering.
Raw HTML features require security review.
```
