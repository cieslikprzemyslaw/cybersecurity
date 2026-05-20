# Cross-Site Scripting (XSS)

This is the third topic in the Key Web Vulnerabilities series.

## Start Here

1. [Overview](./overview.md)
2. [Cheat Sheet](./cheat-sheet.md)
3. [Labs](./labs/README.md)

## Topic Focus

This module covers:

- reflected XSS,
- stored XSS,
- DOM XSS,
- sources and sinks,
- HTML body context,
- HTML attribute context,
- output encoding,
- sanitization,
- safe frontend rendering,
- React-specific XSS considerations,
- developer-focused remediation.

## Labs Covered

| Platform | Lab / Topic | Focus |
|---|---|---|
| TryHackMe | Intro to Cross-site Scripting | XSS fundamentals and basic payload thinking |
| PortSwigger | Reflected XSS into HTML context with nothing encoded | User input reflected directly into HTML |
| PortSwigger | Stored XSS into HTML context with nothing encoded | User input saved and rendered later |
| PortSwigger | DOM XSS in `document.write` sink using `location.search` source | Source → sink flow in browser-side JavaScript |
| PortSwigger | Reflected XSS into attribute with angle brackets HTML-encoded | Attribute context and event handler injection |

## Key Idea

XSS is not just about making `alert(1)` execute. The real lesson is understanding how untrusted data moves into a browser execution context.

```text
user-controlled input → unsafe rendering context → browser treats data as code
```

## Main Frontend Takeaway

As a Frontend Engineer, the most important mental model is:

```text
Where does this data come from?
Where does it go?
Is it rendered as text or interpreted as HTML/JavaScript?
```

Safe rendering is not only a backend concern. Frontend code can introduce XSS through unsafe DOM APIs, raw HTML rendering, risky markdown/rich text handling, or poorly reviewed CMS content rendering.
