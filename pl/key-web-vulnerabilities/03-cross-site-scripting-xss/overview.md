# Cross-Site Scripting (XSS) — Notatki AppSec

> **Ścieżka nauki:** Frontend Engineer → Application Security
> **Źródła:** TryHackMe + PortSwigger Web Security Academy
> **Temat:** Reflected XSS, Stored XSS, DOM XSS, source, sink, context, output encoding, safe rendering
> **Status:** Notatki do nauki, nie pełny walkthrough labów

---

## TL;DR

**XSS dzieje się wtedy, gdy niezaufane dane są renderowane w taki sposób, że przeglądarka może potraktować je jako kod, a nie zwykły tekst.**

Najważniejsze pytanie to nie tylko:

> Czy mogę odpalić JavaScript?

Lepsze pytanie AppSec brzmi:

> Gdzie dane użytkownika wchodzą do aplikacji i gdzie są renderowane?

```text
Untrusted input + unsafe browser context = possible XSS
```

---

## Czym jest XSS?

Cross-Site Scripting to client-side injection issue. Pozwala na wykonanie attacker-controlled JavaScript albo aktywnej treści w przeglądarce innego użytkownika, w kontekście podatnej aplikacji.

W zależności od aplikacji i uprawnień ofiary, XSS może pozwolić na:

- wykonywanie akcji jako ofiara,
- odczyt danych widocznych dla ofiary,
- modyfikację widoku strony,
- przechwytywanie danych wpisywanych przez użytkownika,
- nadużywanie funkcji aplikacji,
- atakowanie adminów, jeśli payload wykonuje się w panelu administracyjnym.

Impact zależy od:

- gdzie payload się wykonuje,
- kto go zobaczy,
- co aplikacja pozwala zrobić temu użytkownikowi,
- czy dostępne są dane wrażliwe albo akcje administracyjne.

---

## Główne typy XSS

### Reflected XSS

Reflected XSS występuje wtedy, gdy dane z aktualnego requestu są umieszczane w natychmiastowej odpowiedzi w niebezpieczny sposób.

Typowy pattern:

```text
GET /search?q=userInput
```

Aplikacja odbija `userInput` w HTML response bez bezpiecznego encode’owania.

Najważniejsza myśl:

```text
request input → immediate response → browser execution
```

Często wykorzystuje się to przez specjalnie przygotowany link.

---

### Stored XSS

Stored XSS występuje wtedy, gdy niezaufane dane są zapisywane przez aplikację i później renderowane użytkownikom w niebezpieczny sposób.

Typowe miejsca:

- komentarze,
- support tickets,
- profile użytkowników,
- reviews,
- wiadomości czatu,
- admin dashboards,
- CMS content.

Najważniejsza myśl:

```text
input saved → later rendered → browser execution for future viewers
```

Stored XSS może być groźniejszy niż reflected XSS, bo ofiara nie musi kliknąć specjalnego linku. Wystarczy, że otworzy stronę, na której payload został zapisany.

---

### DOM XSS

DOM XSS występuje wtedy, gdy JavaScript po stronie przeglądarki bierze dane kontrolowane przez atakującego i zapisuje je do strony przez unsafe API.

Najważniejsza myśl:

```text
browser-controlled source → unsafe JavaScript sink → DOM execution
```

Payload może nie być widoczny w oryginalnej odpowiedzi serwera. Może pojawić się dopiero po wykonaniu JavaScriptu w przeglądarce.

Przykład ryzykownego flow:

```js
const name = new URLSearchParams(window.location.search).get("name");
document.getElementById("welcome").innerHTML = name;
```

Tutaj:

- source: `window.location.search`
- sink: `innerHTML`

Jeśli `name` jest kontrolowane przez atakującego, może to prowadzić do DOM XSS.

---

## Source i Sink

### Source

**Source** to miejsce, z którego pochodzą dane kontrolowane przez użytkownika.

Typowe DOM XSS sources:

```js
location.href
location.search
location.hash
document.referrer
localStorage
sessionStorage
postMessage
```

### Sink

**Sink** to miejsce, w którym dane są używane w potencjalnie niebezpieczny sposób.

Typowe dangerous sinks:

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

Dobre pytanie do code review:

```text
Czy attacker-controlled data może trafić do unsafe sink?
```

---

## XSS Context ma znaczenie

Ten sam payload nie działa wszędzie. XSS zależy od kontekstu, w którym input jest renderowany.

Typowe konteksty:

- HTML body,
- HTML attribute,
- JavaScript string,
- URL,
- CSS,
- DOM po wykonaniu JavaScriptu.

### HTML Body Context

Przykład:

```html
<p>You searched for: USER_INPUT</p>
```

Jeśli aplikacja nie encode’uje outputu, payload może zostać potraktowany jako HTML/aktywny content.

### HTML Attribute Context

Przykład:

```html
<input value="USER_INPUT">
```

Jeśli `<` i `>` są encoded, tworzenie nowego taga może nie działać. Atakujący może próbować wyjść z istniejącego atrybutu i dodać nowy atrybut/event handler.

Lekcja z laba attribute-context:

```text
" zamyka istniejący value attribute
prawdziwy event handler, np. onfocus, może wykonać JavaScript
```

Najważniejsza lekcja to nie konkretny payload, tylko zrozumienie kontekstu i tego, jak przeglądarka parsuje końcowy HTML.

---

## Sanitization vs Output Encoding

### Sanitization

Sanitization usuwa albo neutralizuje niebezpieczną treść.

Cel:

```text
Usunąć unsafe tags, attributes, scripts, event handlers albo dangerous URLs z rich text.
```

Sanitization jest potrzebna, gdy aplikacja naprawdę musi pozwalać na część HTML, np. rich text albo CMS content.

### Output Encoding

Output encoding zamienia specjalne znaki tak, aby przeglądarka wyświetliła je jako tekst, a nie interpretowała jako markup albo kod.

Przykład:

```html
<script>
```

staje się:

```html
&lt;script&gt;
```

Przeglądarka pokazuje tekst, ale go nie wykonuje.

Najważniejsze:

```text
Output encoding musi być context-aware.
```

HTML body, HTML attributes, JavaScript strings, URLs i CSS wymagają innego podejścia.

---

## Ryzykowne wzorce frameworkowe i renderowania

Nowoczesne frameworki zwykle sprawiają, że normalne renderowanie tekstu jest bezpieczniejsze, ale XSS może wrócić, gdy kod omija te domyślne zabezpieczenia.

Szczególnie sprawdzaj:

- raw HTML rendering,
- direct DOM manipulation,
- framework escape/sanitization bypass APIs,
- markdown, CMS albo rich text renderowany jako HTML,
- user-controlled URLs.

Notatki per stack są w [ryzykownych wzorcach frameworkowych](./framework-risky-patterns/README.md).

---

## Co przerobiłem

| Platforma | Lab / temat | Główna lekcja |
|---|---|---|
| TryHackMe | Intro to Cross-site Scripting | Typy XSS, podstawowe payloady, reflected/stored/DOM |
| PortSwigger | Reflected XSS into HTML context with nothing encoded | Input odbity w HTML może się wykonać, jeśli nie jest encoded |
| PortSwigger | Stored XSS into HTML context with nothing encoded | Stored payload może wykonać się później u innych użytkowników |
| PortSwigger | DOM XSS in `document.write` sink using `location.search` source | DOM XSS to source-to-sink flow w JavaScript po stronie przeglądarki |
| PortSwigger | Reflected XSS into attribute with angle brackets HTML-encoded | Payload musi pasować do kontekstu renderowania |

---

## Remediacja dla developerów

Dobre praktyki:

- traktuj wszystkie dane od użytkownika jako niezaufane,
- encode output zależnie od kontekstu,
- unikaj unsafe DOM sinks typu `innerHTML`, jeśli wystarczy renderowanie tekstu,
- używaj `textContent` zamiast `innerHTML` dla zwykłego tekstu,
- unikaj `dangerouslySetInnerHTML`, chyba że naprawdę jest potrzebne,
- sanitizuj rich text/HTML sprawdzonym sanitizerem i strict allowlist,
- waliduj input przy wejściu, ale nie polegaj tylko na walidacji,
- używaj bezpiecznych defaultów frameworka,
- dodaj CSP jako defence-in-depth, ale nie jako jedyny fix,
- przeglądaj dokładnie markdown/CMS/rich text rendering,
- dodaj regression tests sprawdzające, że input wygląda jak tekst, a nie wykonuje się jako kod.

Ryzykowny pattern:

```js
element.innerHTML = userInput;
```

Bezpieczniejszy pattern dla tekstu:

```js
element.textContent = userInput;
```

Bezpieczniejszy pattern w React:

```jsx
<p>{userInput}</p>
```

---

## Checklist do review

Podczas code review albo testowania zapytaj:

- Skąd pochodzi ten input?
- Czy użytkownik może go kontrolować?
- Gdzie jest renderowany?
- Czy jest renderowany jako tekst, HTML, JavaScript, URL czy CSS?
- Czy output encoding jest poprawny dla tego kontekstu?
- Czy używamy raw HTML rendering?
- Czy używamy `innerHTML`, `document.write` albo `dangerouslySetInnerHTML`?
- Czy markdown/rich text/CMS content jest sanitizowany?
- Czy te same dane mogą być wyświetlone później innemu użytkownikowi?
- Czy jest regression test pokazujący, że payload-like input jest wyświetlany bezpiecznie?

---

## OWASP / AppSec mapping

Powiązana kategoria:

- **OWASP Top 10: Injection / XSS-related client-side injection**

Powiązane zasady:

- never trust user input,
- context-aware output encoding,
- safe rendering APIs,
- defence in depth,
- secure defaults,
- avoid unsafe browser sinks.

---

## Najważniejszy wniosek

XSS zależy od kontekstu.

```text
Ten sam input może być bezpieczny w jednym miejscu i niebezpieczny w innym.
```

Jako Frontend Engineer najważniejszy nawyk to śledzenie data flow:

```text
source → transformation → rendering context → browser behaviour
```
