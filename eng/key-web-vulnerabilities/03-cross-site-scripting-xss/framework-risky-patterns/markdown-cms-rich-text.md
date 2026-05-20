# XSS Risks in Markdown, CMS and Rich Text Rendering

## Purpose

Markdown, CMS fields and rich text editors often produce HTML. XSS risk appears when untrusted content is converted to HTML and rendered without safe sanitization.

## Common scenarios

```text
blog comments
CMS rich text fields
support tickets
user profiles
markdown descriptions
product descriptions
documentation pages
admin notes
```

## Risky pattern

```tsx
<div dangerouslySetInnerHTML={{ __html: htmlFromMarkdownOrCms }} />
```

or:

```js
content.innerHTML = htmlFromCms;
```

## Why it is risky

Content from markdown, CMS or rich text may contain:

```text
<script> tags
event handlers such as onerror or onclick
javascript: URLs
iframes
SVG with active content
unsafe attributes
unexpected HTML
```

If rendered directly, this can become stored XSS.

## Safer approach

- Decide whether HTML is truly required.
- If not, render content as plain text.
- If rich text is required, sanitize it before rendering.
- Use a strict allowlist of tags and attributes.
- Remove event handlers.
- Block `javascript:` URLs.
- Restrict iframes, SVG, scripts and dangerous attributes.
- Consider serving user-uploaded content from a separate domain/origin.
- Add CSP as defence-in-depth.

## Markdown warning

Markdown is not automatically safe if raw HTML is allowed.

Review markdown configurations:

```text
Is raw HTML enabled?
Are links sanitized?
Are images restricted?
Are dangerous protocols blocked?
Is output sanitized after conversion?
```

## CMS warning

CMS content may be trusted from an editorial perspective, but it is still content that can become executable in the browser.

This is especially important when:

```text
multiple editors exist
content comes from integrations
content is imported
content fields support raw HTML
content is rendered in different contexts
```

## Review checklist

- Is markdown converted to HTML?
- Is raw HTML allowed?
- Is CMS rich text rendered as HTML?
- Is sanitization applied after conversion?
- Is the sanitizer allowlist strict?
- Are event handlers removed?
- Are dangerous URL schemes blocked?
- Is SVG allowed?
- Is content displayed to other users?
- Is CSP configured as defence-in-depth?

## Developer takeaway

```text
Rich text is still user/content-controlled input. If it becomes HTML, it needs sanitization and context-aware rendering.
```
