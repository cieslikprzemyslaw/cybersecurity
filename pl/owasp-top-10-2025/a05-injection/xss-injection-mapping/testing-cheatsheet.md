# XSS Testing Cheatsheet

> Używaj tylko w autoryzowanych labach i środowiskach testowych. Zacznij od nieszkodliwych markerów i minimalnych proof-of-concept payloads.

## 1. Zidentyfikuj source

Zapisz:

- parametr, pole, header, stored record, CMS field albo browser source,
- request method i content type,
- czy wartość jest reflected, stored, czy przetwarzana tylko w DOM.

## 2. Użyj unikalnego markera tekstowego

```text
XSS_TEST_7F31
```

Szukaj go w:

- raw HTTP response,
- page source,
- rendered DOM,
- attributes,
- inline scripts,
- JavaScript data structures,
- network responses.

## 3. Zidentyfikuj kontekst

Przykłady:

```html
<p>INPUT</p>
<input value="INPUT">
<script>const value = "INPUT";</script>
<a href="INPUT">link</a>
```

Nie wybieraj payloadu, dopóki nie znasz kontekstu.

## 4. Minimalny autoryzowany proof of execution

HTML context:

```html
<img src=x onerror=alert(1)>
```

Attribute-breakout example:

```html
"><svg onload=alert(1)>
```

Użyj najmniejszego payloadu potrzebnego do potwierdzenia browser execution. Nie dodawaj data theft, persistence ani external callbacks, chyba że autoryzowany test jawnie tego wymaga.

## 5. DOM-based review

Śledź:

```text
source → transformation → sink
```

Przykładowe review targets:

```js
location.hash
location.search
window.name
postMessage event data
localStorage
```

oraz:

```js
innerHTML
insertAdjacentHTML
document.write
eval
new Function
```

## 6. React review

Szukaj:

- `dangerouslySetInnerHTML`,
- direct DOM manipulation,
- HTML parsers i rich-text renderers,
- URL values pochodzących od użytkownika albo CMS,
- third-party components przyjmujących raw HTML,
- ręcznego składania stringów dla script albo markup contexts.

Potwierdź, czy HTML jest sanityzowany przed komponentem i czy dangerous protocols oraz attributes są odrzucane.

## 7. Stored XSS checks

Zweryfikuj:

- gdzie wartość jest zapisana,
- które role mogą ją zapisać,
- którzy użytkownicy ją renderują,
- czy preview i live pages zachowują się inaczej,
- czy ta sama wartość jest używana w admin interfaces, emails, PDFs albo innych kanałach.

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

Po poprawce przetestuj:

- oryginalny payload,
- harmless marker,
- istotne alternatywne konteksty,
- stored i reflected paths,
- DOM rendering po client-side navigation,
- preview i production rendering,
- encoded variants, gdy ma to sens,
- regresję w innych komponentach używających tego samego pola.
