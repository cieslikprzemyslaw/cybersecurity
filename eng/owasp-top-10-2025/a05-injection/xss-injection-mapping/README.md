# Cross-Site Scripting Mapping to A05:2025 Injection

## Why XSS belongs in A05

OWASP Top 10:2025 includes Cross-Site Scripting under A05 Injection. XSS occurs when attacker-controlled data reaches a browser execution context without the correct context-aware encoding, sanitisation, or safe DOM handling.

```text
attacker-controlled data
→ application output or DOM operation
→ browser interprets data as active content
→ script or attacker-controlled behaviour executes
```

The browser is the interpreter.

## Related internal XSS notes

- [Key Web Vulnerabilities: Cross-Site Scripting](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [XSS overview](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md)
- [XSS cheat sheet](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/cheat-sheet.md)
- [XSS framework risky patterns](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/framework-risky-patterns/README.md)
- [XSS labs](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/labs/README.md)

## Main XSS types

### Reflected XSS

The malicious value is included in the immediate response, commonly through a query parameter, form input, header, or error message.

### Stored XSS

The malicious value is saved and later rendered to other users. Common storage locations include CMS fields, comments, profiles, support tickets, database records, and rich-text content.

### DOM-based XSS

Client-side JavaScript reads attacker-controlled data from a source and writes it to a dangerous sink without safe handling. The vulnerability may exist entirely in browser-side code.

## Source-to-sink mental model

### Common sources

- `location.search`,
- `location.hash`,
- `document.URL`,
- `document.referrer`,
- `postMessage` data,
- local or session storage,
- API and CMS responses,
- user profile or form data.

### Dangerous sinks

- `innerHTML`,
- `outerHTML`,
- `insertAdjacentHTML`,
- `document.write`,
- string-based `setTimeout` or `setInterval`,
- `eval` and `Function`,
- unsafe URL assignment,
- React `dangerouslySetInnerHTML`,
- third-party HTML rendering components.

## Context matters

The correct defence depends on where the value is placed:

- HTML text context,
- HTML attribute context,
- JavaScript context,
- CSS context,
- URL context,
- raw HTML or rich-text context.

One generic escaping function is not correct for every context.

## XSS versus SSTI

| Question | XSS | SSTI |
|---|---|---|
| Main interpreter | Browser / JavaScript engine | Server-side template engine |
| Processing time | After the response reaches the browser | Before the response is generated |
| Typical evidence | Script execution or another browser-side effect | Template expression evaluated into a server-side result |
| Main target | User session, browser context, and page-accessible data | Server application, template context, files, processes, or secrets |
| Primary fix | Context-aware output encoding, safe DOM APIs, and HTML sanitisation where raw HTML is required | Keep template source static and pass user input only as template data |

HTML rendering alone does not prove SSTI. Server-side expression evaluation does not prove XSS. The evidence must match the interpreter and processing stage.

## React perspective

React escapes values rendered through normal JSX expressions:

```tsx
<p>{userControlledValue}</p>
```

This protection can be bypassed when developers use escape hatches or unsafe APIs, especially:

```tsx
<div dangerouslySetInnerHTML={{ __html: cmsHtml }} />
```

When HTML rendering is a real requirement:

- sanitise using a maintained HTML sanitiser,
- define an allowlist of tags and attributes,
- validate URLs and protocols,
- avoid allowing scripts, event handlers, dangerous elements, and unsafe URI schemes,
- sanitise at the appropriate trust boundary,
- test the final rendered DOM.

## CMS and rich-text risk

CMS content is not automatically trusted. Consider:

- who can create or edit content,
- whether content can be imported,
- whether compromised accounts can add active content,
- whether previews and live rendering use the same sanitisation,
- whether stored HTML is sanitised on write, render, or both,
- whether different frontend components handle the same field differently.

## Evidence

Strong evidence should show:

1. the controlled source,
2. the exact rendering or DOM path,
3. the execution context,
4. repeatable script execution or another meaningful browser-side effect,
5. which users or pages are affected.

Do not describe every `alert(1)` as account takeover. Explain the demonstrated execution and then state realistic possible impacts such as session actions, data access available to the page, UI manipulation, phishing, or actions performed as the victim.

## Prevention

- use framework auto-escaping,
- apply context-aware output encoding,
- sanitise HTML only when raw HTML is required,
- avoid dangerous DOM sinks,
- validate URL schemes and destinations,
- avoid inline scripts and string-to-code APIs,
- use Content Security Policy as defence in depth, not the only defence,
- keep sanitiser and framework dependencies updated,
- add reflected, stored, and DOM XSS regression tests.

## Regression tests

- The original reflected payload no longer executes.
- Stored content is safe for all affected roles and pages.
- DOM-based source-to-sink paths no longer reach unsafe sinks.
- Normal JSX rendering remains escaped.
- Sanitised rich text removes dangerous tags, event handlers, and unsafe URL schemes.
- Preview and live CMS rendering use equivalent protections.
- Related components consuming the same field remain safe.
- CSP reports are reviewed as defence-in-depth signals, not treated as proof that rendering is fixed.

For the detailed workflow, see [testing-cheatsheet.md](testing-cheatsheet.md).

## References

- [OWASP A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
- [OWASP Cross Site Scripting](https://owasp.org/www-community/attacks/xss/)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
