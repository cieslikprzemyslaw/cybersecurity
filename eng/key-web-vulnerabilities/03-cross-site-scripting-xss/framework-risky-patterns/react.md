# XSS Risky Patterns in React

## Purpose

React is usually safe by default when rendering values through normal JSX. The risk increases when developers bypass JSX escaping or inject raw HTML.

## Safe by default

React escapes values rendered in JSX.

```tsx
const userInput = "<img src=x onerror=alert(1)>";

return <p>{userInput}</p>;
```

In this case, React treats `userInput` as text, not as executable HTML.

## Risky pattern: `dangerouslySetInnerHTML`

```tsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

## Why it is risky

`dangerouslySetInnerHTML` renders raw HTML into the DOM. If `userInput` comes from a user, CMS, markdown, API response, support ticket, comment, or profile field, it may lead to XSS.

The name is intentionally warning the developer that this bypasses React's normal escaping.

## Safer alternatives

Prefer normal JSX rendering:

```tsx
<p>{userInput}</p>
```

If raw HTML is truly required:

- sanitize the HTML before rendering,
- use a strict allowlist of allowed tags and attributes,
- block scripts, event handlers, dangerous URLs, and unexpected attributes,
- avoid rendering user-controlled HTML if there is no clear business need.

## Other React XSS review points

Review these patterns carefully:

```text
dangerouslySetInnerHTML
html-react-parser
markdown rendered as HTML
CMS rich text rendered as HTML
manual DOM manipulation with innerHTML
user-controlled href or src
javascript: URLs
third-party widgets
```

## URL and link warning

Even if React escapes text, developers still need to validate URLs.

Risky idea:

```tsx
<a href={userControlledUrl}>Open</a>
```

If `userControlledUrl` can be `javascript:...`, this may become dangerous depending on browser and rendering behaviour.

Safer approach:

- allow only expected protocols such as `https:`,
- validate URLs on the server and client where appropriate,
- avoid directly trusting URLs from user input.

## Review checklist

- Is raw HTML rendering used?
- Where does the HTML come from?
- Is the content trusted?
- Is a sanitizer used?
- Is the sanitizer configured with a strict allowlist?
- Are event handlers blocked?
- Are `javascript:` URLs blocked?
- Is CMS or markdown content being rendered?
- Can the same result be achieved with normal JSX?

## Developer takeaway

```text
React helps prevent XSS by default, but only if developers keep data inside normal JSX rendering.
```

Raw HTML rendering should always trigger a security review.
