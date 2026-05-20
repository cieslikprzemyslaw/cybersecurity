# Lab Summary: TryHackMe — Intro to Cross-site Scripting

> **Platform:** TryHackMe  
> **Topic:** XSS fundamentals  
> **Status:** Completed  
> **Note type:** Learning summary, not a full walkthrough

---

## What I Practised

I reviewed the basic types of XSS and how user-controlled input can be reflected, stored, or handled in browser-side JavaScript.

The room helped build the mental model for:

- reflected XSS,
- stored XSS,
- DOM XSS,
- payloads as proof of concept,
- why context matters,
- why browser execution is the core issue.

---

## Key Concepts

### Reflected XSS

Input from the current request is included in the immediate response in an unsafe way.

### Stored XSS

Input is saved by the application and later displayed to users in an unsafe way.

### DOM XSS

Client-side JavaScript takes attacker-controlled data and writes it into the DOM using an unsafe sink.

---

## What I Learned

The most useful lesson was that XSS is not only about memorising payloads. The important part is understanding where the input appears and how the browser interprets it.

```text
source → context → sink/rendering → browser behaviour
```

---

## Developer Takeaway

Developers should avoid rendering untrusted data as HTML unless it is genuinely required and safely sanitized.

For normal text, render as text:

```js
element.textContent = userInput;
```

or in React:

```jsx
<p>{userInput}</p>
```

---

## Main Takeaway

```text
XSS happens when untrusted data reaches a browser context where it can be interpreted as code.
```
