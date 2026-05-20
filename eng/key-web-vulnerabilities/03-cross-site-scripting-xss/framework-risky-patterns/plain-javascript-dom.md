# XSS Risky Patterns in Plain JavaScript and DOM APIs

## Purpose

DOM XSS often happens when JavaScript takes attacker-controlled data and writes it into the page using unsafe DOM APIs.

## Common sources

User-controlled data can come from:

```text
location.search
location.hash
location.href
document.referrer
localStorage
sessionStorage
postMessage
API responses
form inputs
```

## Dangerous sinks

Review these carefully:

```js
element.innerHTML = userInput;
element.outerHTML = userInput;
document.write(userInput);
element.insertAdjacentHTML("beforeend", userInput);
eval(userInput);
new Function(userInput);
setTimeout(userInput, 1000);
setInterval(userInput, 1000);
```

## Why it is risky

These APIs can treat strings as HTML or JavaScript. If attacker-controlled data reaches one of these sinks, the browser may execute it.

## Safer alternatives

For text:

```js
element.textContent = userInput;
```

For attributes:

```js
element.setAttribute("title", safeValue);
```

For DOM creation:

```js
const p = document.createElement("p");
p.textContent = userInput;
container.appendChild(p);
```

For HTML that must be rendered:

- sanitize before rendering,
- use strict allowlists,
- block scripts, event handlers, dangerous attributes and URLs,
- avoid raw HTML if possible.

## Example review

Risky:

```js
const name = new URLSearchParams(location.search).get("name");
document.getElementById("welcome").innerHTML = name;
```

Safer:

```js
const name = new URLSearchParams(location.search).get("name");
document.getElementById("welcome").textContent = name;
```

## Review checklist

- What is the source of the data?
- Is the data attacker-controlled?
- Does it reach `innerHTML`, `document.write`, `eval`, or similar sinks?
- Can `textContent` be used instead?
- Is HTML rendering actually required?
- Is sanitization applied if HTML rendering is required?
- Are URL schemes validated?

## Developer takeaway

```text
DOM XSS is often source-to-sink: attacker-controlled data flows into an unsafe DOM API.
```
