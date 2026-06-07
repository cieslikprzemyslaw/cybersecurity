# XSS Testing Cheatsheet

> Use only in authorised labs and test environments. Start with harmless markers and minimal proof-of-concept payloads.

## 1. Identify the source

Record:

- parameter, field, header, stored record, CMS field, or browser source,
- request method and content type,
- whether the value is reflected, stored, or processed only in the DOM.

## 2. Use a unique text marker

```text
XSS_TEST_7F31
```

Search for it in:

- raw HTTP response,
- page source,
- rendered DOM,
- attributes,
- inline scripts,
- JavaScript data structures,
- network responses.

## 3. Identify the context

Examples:

```html
<p>INPUT</p>
<input value="INPUT">
<script>const value = "INPUT";</script>
<a href="INPUT">link</a>
```

Do not select a payload until the context is known.

## 4. Minimal authorised proof of execution

HTML context:

```html
<img src=x onerror=alert(1)>
```

Attribute-breakout example:

```html
"><svg onload=alert(1)>
```

Use only the smallest payload needed to prove browser execution. Do not add data theft, persistence, or external callbacks unless the authorised test explicitly requires them.

## 5. DOM-based review

Trace:

```text
source → transformation → sink
```

Example review targets:

```js
location.hash
location.search
window.name
postMessage event data
localStorage
```

and:

```js
innerHTML
insertAdjacentHTML
document.write
eval
new Function
```

## 6. React review

Search for:

- `dangerouslySetInnerHTML`,
- direct DOM manipulation,
- HTML parsers and rich-text renderers,
- URL values derived from user or CMS data,
- third-party components that accept raw HTML,
- manual string construction for script or markup contexts.

Confirm whether HTML is sanitised before reaching the component and whether dangerous protocols and attributes are rejected.

## 7. Stored XSS checks

Verify:

- where the value is stored,
- which roles can write it,
- which users render it,
- whether preview and live pages behave differently,
- whether the same value is reused in admin interfaces, emails, PDFs, or other channels.

## 8. Evidence template

```text
Controlled source:
Storage location, if any:
Rendering location:
Execution context:
Payload or marker:
Observed browser behaviour:
Affected role or user:
Repeatability:
Confirmed impact:
Possible additional impact:
```

## 9. Remediation verification

After the fix, test:

- the original payload,
- a harmless marker,
- relevant alternate contexts,
- stored and reflected paths,
- DOM rendering after client-side navigation,
- preview and production rendering,
- encoded variants where appropriate,
- regression in other components using the same field.
