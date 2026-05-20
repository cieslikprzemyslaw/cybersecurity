# XSS Risky Patterns in Server-Side Templates

## Purpose

Server-side templates generate HTML on the backend. XSS can happen when user-controlled data is rendered as raw HTML instead of escaped text.

## What are server-side templates?

Server-side templates combine HTML with data.

Examples include:

```text
PHP templates
Twig
Blade
Jinja2
Django templates
EJS
Handlebars
Pug
Razor
Thymeleaf
```

The backend produces a complete HTML response and sends it to the browser.

## Common risky pattern

Many template engines escape values by default, but provide a way to render raw HTML.

Review these patterns:

```text
PHP: echo $userInput without htmlspecialchars
Twig: {{ value|raw }}
Jinja / Django: {{ value|safe }}
EJS: <%- value %>
Handlebars: {{{ value }}}
Razor: @Html.Raw(value)
Thymeleaf: th:utext
```

## Why it is risky

Raw output tells the template engine not to escape HTML. If the value is user-controlled, the browser may execute injected markup or JavaScript.

## Safer alternatives

Use escaped output by default.

Examples:

```text
Twig: {{ value }}
Jinja / Django: {{ value }}
EJS: <%= value %>
Handlebars: {{ value }}
Razor: @value
Thymeleaf: th:text
```

If raw HTML is required:

- sanitize before rendering,
- use strict allowlists,
- block scripts and event handlers,
- validate URLs,
- document why raw output is needed.

## Context matters

Escaping must match the context:

```text
HTML body
HTML attribute
JavaScript string
URL
CSS
```

Escaping for one context may not be safe for another context.

## Review checklist

- Is autoescaping enabled?
- Is raw output used?
- Where does the raw value come from?
- Is the value user-controlled or CMS-controlled?
- Is sanitization applied?
- Is output inserted into HTML body, attribute, JavaScript, URL, or CSS?
- Could normal escaped output be used instead?

## Developer takeaway

```text
Autoescaping helps, but raw output features bypass template protections.
```

Any raw output feature should be treated as a security review point.
