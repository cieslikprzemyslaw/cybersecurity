# XSS Cheat Sheet — Praktyczny workflow

> Ta ściąga jest tylko do legalnych labów, lokalnych środowisk i autoryzowanych testów.

---

## Główne pytanie

```text
Czy attacker-controlled data może trafić do browser execution context bez bezpiecznego encodingu albo sanitizacji?
```

---

## 1. Znajdź input points

Szukaj miejsc, gdzie użytkownik może wysłać albo kontrolować dane:

- search boxes,
- URL query parameters,
- URL path values,
- comments,
- profile fields,
- support tickets,
- chat messages,
- CMS/rich text fields,
- file names,
- uploadowane SVG/HTML-like content,
- headers typu Referer albo User-Agent w niektórych przypadkach,
- DOM sources typu `location.search` albo `location.hash`.

---

## 2. Najpierw użyj unikalnego test stringa

Zanim użyjesz payloadu, zacznij od prostego markera:

```text
xsstest123
```

Potem sprawdź:

- Czy pojawia się w HTTP response?
- Czy pojawia się w DOM po wykonaniu JavaScriptu?
- Czy jest zapisany i pokazany później?
- Gdzie dokładnie się pojawia?

---

## 3. Rozpoznaj context

Context decyduje o strategii testowania.

| Context | Przykład | Co sprawdzić |
|---|---|---|
| HTML body | `<p>USER_INPUT</p>` | Czy output jest HTML-encoded? |
| HTML attribute | `<input value="USER_INPUT">` | Czy cudzysłów pozwala wyjść z atrybutu? |
| JavaScript string | `var x = 'USER_INPUT'` | Czy string jest bezpiecznie escaped? |
| URL | `<a href="USER_INPUT">` | Czy dangerous schemes są blokowane? |
| DOM sink | `innerHTML = value` | Czy source trafia do unsafe sink? |

---

## 4. Reflected XSS workflow

1. Wyślij request z unikalnym markerem.
2. Sprawdź, czy marker pojawia się w natychmiastowej odpowiedzi.
3. Rozpoznaj context.
4. Sprawdź, czy przeglądarka interpretuje wartość jako kod.
5. Potwierdzaj exploitability tylko w legalnym labie/scope.
6. Opisz root cause i remediation.

Kluczowe pytanie:

```text
Czy request input wraca od razu w response bez safe output encoding?
```

---

## 5. Stored XSS workflow

1. Wyślij marker do pola, które zapisuje dane.
2. Wejdź ponownie na stronę, gdzie dane są wyświetlane.
3. Sprawdź, czy wartość renderuje się innym użytkownikom albo adminom.
4. Rozpoznaj context.
5. Bezpiecznie sprawdź w labie, czy code execution jest możliwe.
6. Opisz, kto byłby dotknięty problemem.

Kluczowe pytanie:

```text
Czy niezaufane dane są zapisane i później renderowane w unsafe way?
```

---

## 6. DOM XSS workflow

Szukaj source-to-sink flow.

Typowe sources:

```js
location.search
location.hash
location.href
document.referrer
localStorage
sessionStorage
postMessage
```

Typowe sinks:

```js
innerHTML
outerHTML
document.write()
insertAdjacentHTML()
eval()
new Function()
setTimeout("string")
setInterval("string")
```

Workflow:

1. Dodaj marker do URL parameter albo hash.
2. Użyj DevTools, żeby znaleźć marker w DOM.
3. Sprawdź JavaScript, który obsługuje wartość.
4. Rozpoznaj source i sink.
5. Zamień unsafe rendering na safe API, jeśli to możliwe.

Kluczowe pytanie:

```text
Czy browser-side JavaScript przenosi attacker-controlled data do unsafe sink?
```

---

## 7. Przypomnienie: attribute context

Jeśli input pojawia się tutaj:

```html
<input value="USER_INPUT">
```

a angle brackets są encoded, payload z nowym tagiem może nie działać.

Lekcja z laba attribute-context:

```text
Wyjdź z aktualnego atrybutu, potem dodaj prawdziwy event handler.
```

Nie ucz się tylko jednego payloadu na pamięć. Zrozum, jak przeglądarka sparsuje końcowy HTML.

---

## 8. React review checklist

Sprawdź:

- `dangerouslySetInnerHTML`,
- `innerHTML` / `outerHTML`,
- `document.write`,
- raw HTML z CMS/API,
- markdown rendering z raw HTML,
- rich text editors,
- `html-react-parser` albo podobne biblioteki,
- third-party components renderujące HTML,
- user-controlled URLs w linkach albo redirectach.

Bezpieczniejszy default:

```jsx
<p>{userInput}</p>
```

Ryzykowny pattern:

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

---

## 9. Remediation quick reference

| Problem | Lepsze podejście |
|---|---|
| Plain text renderowany przez `innerHTML` | Użyj `textContent` albo safe JSX rendering |
| User input w HTML body | HTML output encode |
| User input w HTML attribute | Attribute encode i bezpieczne quote |
| User input w JavaScript string | JavaScript string encoding / unikaj inline JS |
| User-provided HTML jest wymagany | Sanitizer ze strict allowlist |
| CMS/rich text rendering | Sanitizuj i ogranicz allowed tags/attributes |
| Zmniejszenie impactu XSS | CSP jako defence-in-depth |

---

## 10. Regression test ideas

Dodaj testy sprawdzające, że:

- payload-like input wyświetla się jako tekst,
- komentarze nie wykonują JavaScriptu,
- search terms są encoded w HTML responses,
- sanitizer usuwa scripts i event handlers,
- `dangerouslySetInnerHTML` nie jest używane z niezaufanymi danymi,
- DOM sources nie trafiają do unsafe sinks,
- uploadowane SVG/HTML nie są renderowane niebezpiecznie.

---

## Najważniejszy wniosek

```text
Nie pytaj tylko “czy alert działa?”
Pytaj “jaki jest context i czy niezaufane dane są interpretowane jako kod?”
```
