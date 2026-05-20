# XSS Risky Patterns in Vue

## Purpose

Vue escapes normal text interpolation by default. XSS risk increases when developers render raw HTML or trust dynamic attributes such as URLs.

## Safe by default

```vue
<p>{{ userInput }}</p>
```

Vue treats this as text and escapes HTML characters.

## Risky pattern: `v-html`

```vue
<div v-html="userInput"></div>
```

## Why it is risky

`v-html` renders raw HTML. If `userInput` comes from a user, CMS, markdown, API response, comment, profile field, or ticket, it may lead to XSS.

`v-html` should be treated similarly to React's `dangerouslySetInnerHTML`.

## Risky pattern: dynamic URLs

```vue
<a :href="userControlledUrl">Open</a>
```

## Why it is risky

If the URL is user-controlled, it may use an unsafe scheme such as `javascript:` or point to unexpected content.

## Safer alternatives

- Use normal interpolation for text.
- Avoid `v-html` for user-controlled content.
- Sanitize rich text before rendering.
- Allow only expected tags and attributes.
- Validate URLs and restrict allowed protocols.

## Review checklist

- Is `v-html` used?
- Where does the HTML come from?
- Is the HTML sanitized?
- Is the sanitizer configured with an allowlist?
- Are dynamic `href` or `src` values user-controlled?
- Are protocols restricted to safe values such as `https:`?
- Is raw CMS/markdown content being rendered?

## Developer takeaway

```text
Vue is safe for normal text interpolation, but v-html renders raw HTML and requires security review.
```
