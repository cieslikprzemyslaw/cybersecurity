# Cross-Site Scripting Mapping to A05:2025 Injection

## Dlaczego XSS należy do A05

OWASP Top 10:2025 umieszcza Cross-Site Scripting w A05 Injection. XSS występuje wtedy, gdy dane kontrolowane przez atakującego trafiają do kontekstu wykonania przeglądarki bez poprawnego context-aware encoding, sanitisation albo safe DOM handling.

```text
attacker-controlled data
→ application output or DOM operation
→ browser interprets data as active content
→ script or attacker-controlled behaviour executes
```

Interpreterem jest przeglądarka.

## Powiązane wewnętrzne notatki XSS

- [Key Web Vulnerabilities: Cross-Site Scripting](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/README.md)
- [XSS overview](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/overview.md)
- [XSS cheat sheet](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/cheat-sheet.md)
- [Ryzykowne wzorce frameworkowe XSS](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/framework-risky-patterns/README.md)
- [Laby XSS](../../../key-web-vulnerabilities/03-cross-site-scripting-xss/labs/README.md)

## Główne typy XSS

### Reflected XSS

Złośliwa wartość jest włączana do natychmiastowej odpowiedzi, często przez query parameter, form input, header albo error message.

### Stored XSS

Złośliwa wartość jest zapisana i później renderowana innym użytkownikom. Typowe miejsca zapisu to CMS fields, comments, profiles, support tickets, database records i rich-text content.

### DOM-based XSS

Client-side JavaScript czyta dane kontrolowane przez atakującego ze źródła i zapisuje je do niebezpiecznego sink bez bezpiecznej obsługi. Podatność może istnieć całkowicie po stronie przeglądarki.

## Source-to-sink mental model

### Typowe sources

- `location.search`,
- `location.hash`,
- `document.URL`,
- `document.referrer`,
- `postMessage` data,
- local/session storage,
- API i CMS responses,
- user profile albo form data.

### Niebezpieczne sinks

- `innerHTML`,
- `outerHTML`,
- `insertAdjacentHTML`,
- `document.write`,
- string-based `setTimeout` albo `setInterval`,
- `eval` i `Function`,
- unsafe URL assignment,
- React `dangerouslySetInnerHTML`,
- third-party HTML rendering components.

## Kontekst ma znaczenie

Poprawna obrona zależy od miejsca wstawienia wartości:

- HTML text context,
- HTML attribute context,
- JavaScript context,
- CSS context,
- URL context,
- raw HTML albo rich-text context.

Jedna generyczna funkcja escape nie jest poprawna dla każdego kontekstu.

## XSS a SSTI

| Pytanie | XSS | SSTI |
|---|---|---|
| Główny interpreter | Przeglądarka / JavaScript engine | Server-side template engine |
| Moment przetwarzania | Po dotarciu odpowiedzi do przeglądarki | Przed wygenerowaniem odpowiedzi |
| Typowy dowód | Script execution albo inny browser-side effect | Wyrażenie szablonu ocenione do server-side wyniku |
| Główny cel | Sesja użytkownika, browser context i dane dostępne stronie | Aplikacja serwerowa, template context, pliki, procesy albo sekrety |
| Główna poprawka | Context-aware output encoding, safe DOM APIs i HTML sanitisation, gdy raw HTML jest wymagany | Statyczny template source i input użytkownika przekazywany tylko jako template data |

Samo renderowanie HTML nie dowodzi SSTI. Server-side expression evaluation nie dowodzi XSS. Dowód musi pasować do interpretera i etapu przetwarzania.

## Perspektywa React

React escapuje wartości renderowane przez normalne JSX expressions:

```tsx
<p>{userControlledValue}</p>
```

Tę ochronę można ominąć przez escape hatches albo unsafe APIs, szczególnie:

```tsx
<div dangerouslySetInnerHTML={{ __html: cmsHtml }} />
```

Gdy renderowanie HTML jest realnym wymaganiem:

- sanityzuj maintained HTML sanitiserem,
- zdefiniuj allowlist tagów i atrybutów,
- waliduj URL-e i protokoły,
- nie pozwalaj na scripts, event handlers, dangerous elements i unsafe URI schemes,
- sanityzuj na właściwej granicy zaufania,
- testuj finalny rendered DOM.

## CMS i rich-text risk

CMS content nie jest automatycznie zaufany. Sprawdź:

- kto może tworzyć albo edytować treść,
- czy treść może być importowana,
- czy przejęte konto może dodać active content,
- czy preview i live rendering używają tej samej sanitisation,
- czy stored HTML jest sanityzowany on write, on render, albo w obu miejscach,
- czy różne frontend components obsługują to samo pole inaczej.

## Evidence

Mocny dowód powinien pokazywać:

1. kontrolowane source,
2. dokładną ścieżkę rendering albo DOM,
3. execution context,
4. powtarzalne script execution albo inny meaningful browser-side effect,
5. których użytkowników albo stron dotyczy problem.

Nie opisuj każdego `alert(1)` jako account takeover. Wyjaśnij pokazane execution, a potem realistyczne skutki: session actions, data access dostępny dla strony, UI manipulation, phishing albo działania wykonane jako ofiara.

## Prevention

- używaj framework auto-escaping,
- stosuj context-aware output encoding,
- sanityzuj HTML tylko gdy raw HTML jest wymagany,
- unikaj dangerous DOM sinks,
- waliduj URL schemes i destinations,
- unikaj inline scripts i string-to-code APIs,
- używaj Content Security Policy jako defence in depth,
- aktualizuj sanitiser i framework dependencies,
- dodaj reflected, stored i DOM XSS regression tests.

## Testy regresji

- Oryginalny reflected payload już się nie wykonuje.
- Stored content jest bezpieczny dla wszystkich affected roles i pages.
- DOM-based source-to-sink paths nie trafiają już do unsafe sinks.
- Normalne JSX rendering pozostaje escaped.
- Sanitised rich text usuwa dangerous tags, event handlers i unsafe URL schemes.
- Preview i live CMS rendering używają równoważnych ochron.
- Powiązane komponenty konsumujące to samo pole pozostają bezpieczne.
- CSP reports są analizowane jako defence-in-depth signals, a nie jako dowód naprawy renderowania.

Szczegółowy workflow jest w [testing-cheatsheet.md](testing-cheatsheet.md).

## References

- [OWASP A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
- [OWASP Cross Site Scripting](https://owasp.org/www-community/attacks/xss/)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
