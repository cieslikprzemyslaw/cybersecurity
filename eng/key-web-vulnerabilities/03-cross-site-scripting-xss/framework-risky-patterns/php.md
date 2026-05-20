# XSS Risky Patterns in PHP

## Purpose

PHP often renders server-side HTML. XSS can happen when user-controlled data is echoed into a page without context-aware escaping.

## Risky pattern: echoing user input directly

```php
<h1>Hello, <?php echo $_GET['name']; ?></h1>
```

## Why it is risky

If the user controls `name`, they can inject HTML or JavaScript into the response.

Example output:

```html
<h1>Hello, <script>alert(1)</script></h1>
```

The browser may execute the script.

## Safer alternative: HTML escaping

```php
<h1>Hello, <?php echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8'); ?></h1>
```

## Why this helps

`htmlspecialchars` converts special HTML characters into safe text representations.

For example:

```text
< becomes &lt;
> becomes &gt;
" becomes &quot;
' becomes &#039; when ENT_QUOTES is used
```

This helps the browser display user input as text instead of interpreting it as HTML.

## Attribute context warning

HTML body context and attribute context are different.

Risky:

```php
<input value="<?php echo $_GET['name']; ?>">
```

Safer:

```php
<input value="<?php echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8'); ?>">
```

`ENT_QUOTES` is important because quotes can break out of attributes.

## JavaScript context warning

Do not directly inject user input into JavaScript.

Risky:

```php
<script>
  const name = "<?php echo $_GET['name']; ?>";
</script>
```

Safer approaches:

- avoid inline JavaScript with user input,
- pass data as JSON using safe encoding,
- use framework-safe data passing mechanisms,
- apply context-appropriate encoding.

## Review checklist

- Is `$_GET`, `$_POST`, `$_REQUEST`, or database content echoed directly?
- Is output escaped with `htmlspecialchars`?
- Is `ENT_QUOTES` used for attribute contexts?
- Is UTF-8 specified?
- Is user input inserted into JavaScript?
- Is raw HTML really required?
- Is templating autoescape enabled?

## Developer takeaway

```text
In PHP, echoing user-controlled data without escaping is a common XSS risk.
```

Use context-aware escaping, and avoid raw output unless the data is trusted and sanitized.
