# XSS Risky Patterns in Angular

## Purpose

Angular includes built-in protections and sanitization for many template contexts. XSS risk increases when developers bypass Angular's security model or use raw DOM APIs.

## Safe by default

Angular template binding normally escapes or sanitizes values depending on the context.

```html
<p>{{ userInput }}</p>
```

This treats the value as text.

## Risky patterns: bypassing Angular security

Review any use of:

```ts
bypassSecurityTrustHtml()
bypassSecurityTrustUrl()
bypassSecurityTrustResourceUrl()
bypassSecurityTrustScript()
bypassSecurityTrustStyle()
```

## Why it is risky

Methods with `bypassSecurityTrust...` tell Angular to trust a value that it would otherwise protect.

This can be valid for very controlled content, but it is risky when the value comes from users, CMS, markdown, API responses, comments, tickets, or external systems.

## Risky pattern: raw DOM APIs

```ts
elementRef.nativeElement.innerHTML = userInput;
```

or:

```ts
document.write(userInput);
```

## Why it is risky

Direct DOM manipulation can bypass Angular's template protections. If attacker-controlled input reaches `innerHTML`, it may become executable HTML.

## Safer alternatives

- Prefer Angular template interpolation for text.
- Avoid `bypassSecurityTrust...` unless there is a strong reason.
- Sanitize untrusted HTML before rendering.
- Use strict allowlists for rich text.
- Validate URLs and allowed protocols.

## Review checklist

- Is `bypassSecurityTrust...` used?
- Why is it needed?
- Is the value fully trusted?
- Is it user-controlled, CMS-controlled, or external?
- Is raw DOM manipulation used?
- Is `innerHTML` assigned directly?
- Are URLs validated?
- Is the content sanitized with a strict allowlist?

## Developer takeaway

```text
If Angular code says “bypass security”, treat it as a security review point.
```

Angular helps by default, but bypass APIs and direct DOM manipulation can reintroduce XSS risk.
