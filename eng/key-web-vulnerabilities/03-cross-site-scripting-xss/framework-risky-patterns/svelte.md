# XSS Risky Patterns in Svelte

## Purpose

Svelte escapes normal values by default. XSS risk increases when developers use raw HTML rendering.

## Safe by default

```svelte
<p>{userInput}</p>
```

Svelte treats `userInput` as text and escapes HTML characters.

## Risky pattern: `{@html ...}`

```svelte
{@html userInput}
```

## Why it is risky

`{@html}` renders raw HTML. If `userInput` comes from users, CMS, markdown, API responses, comments, profiles, or external systems, it may lead to XSS.

## Safer alternatives

- Use normal Svelte text rendering.
- Avoid raw HTML unless there is a clear business need.
- Sanitize untrusted HTML before rendering.
- Use a strict allowlist of safe tags and attributes.
- Validate dynamic URLs.

## Review checklist

- Is `{@html}` used?
- Where does the HTML come from?
- Is the content user-controlled or externally controlled?
- Is sanitization applied?
- Are event handlers and scripts blocked?
- Are unsafe URL schemes blocked?
- Can normal text rendering be used instead?

## Developer takeaway

```text
In Svelte, {@html} means raw HTML rendering and should trigger a security review.
```
