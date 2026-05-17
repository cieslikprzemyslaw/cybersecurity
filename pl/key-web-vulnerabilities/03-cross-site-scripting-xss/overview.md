# Cross-Site Scripting (XSS) - notatki AppSec

> Ścieżka nauki: Frontend Engineer -> Application Security  
> Temat: Reflected XSS, Stored XSS, DOM XSS, source, sink, output encoding, safe rendering  
> Status: notatki edukacyjne, nie pełny walkthrough labów

## TL;DR

XSS nie polega tylko na wstrzyknięciu `<script>`.

Prawdziwy problem polega na tym, że niezaufane dane są renderowane w miejscu, w którym przeglądarka może potraktować je jako kod.

```text
Niezaufane dane + niebezpieczny kontekst renderowania = potencjalny XSS
```

Z perspektywy frontend developera najważniejsze pytanie brzmi:

> Czy ta wartość jest renderowana jako bezpieczny tekst, czy przeglądarka może zinterpretować ją jako HTML/JavaScript?

## Czym jest XSS?

Cross-Site Scripting występuje wtedy, gdy aplikacja umieszcza niezaufane dane na stronie w sposób, który pozwala przeglądarce potraktować je jako wykonywalny kod.

Problemem nie jest sam fakt, że użytkownik może wpisać coś niebezpiecznego. Użytkownik zawsze może kontrolować input.

Problemem jest to, że aplikacja nie obsłużyła tych danych bezpiecznie przed renderowaniem.

## Reflected XSS

Reflected XSS występuje, gdy input z requestu jest od razu zwracany w response.

Przykład:

```http
GET /search?q=test
```

Jeśli wartość `q` trafia do HTML bez poprawnego encodingu, może stać się podatna zależnie od kontekstu.

Pytania podczas testowania:

- Który parametr kontroluję?
- Czy moja wartość pojawia się w response?
- Czy jest zakodowana, czy zwrócona raw?
- W jakim kontekście jest odbita?
- Czy ofiara musiałaby kliknąć spreparowany link?

## Stored XSS

Stored XSS występuje, gdy złośliwy input jest zapisany przez aplikację i później wyświetlany innym użytkownikom.

Typowe miejsca:

- komentarze,
- profile użytkowników,
- support tickets,
- chat messages,
- CMS content,
- product reviews,
- admin panels.

Stored XSS może mieć większy impact niż reflected XSS, bo payload może wykonać się u każdego użytkownika, który otworzy daną stronę.

Ważne pytanie:

> Kto zobaczy tę zapisaną treść i jakie ma uprawnienia?

## DOM XSS

DOM XSS występuje, gdy JavaScript w przeglądarce bierze dane kontrolowane przez atakującego i zapisuje je do strony przez niebezpieczne API.

Schemat:

```text
attacker-controlled source -> unsafe sink -> browser execution
```

Przykład:

```js
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('welcome').innerHTML = name;
```

Tutaj:

- source: `window.location.search`,
- sink: `innerHTML`.

Bezpieczniejsza wersja dla zwykłego tekstu:

```js
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('welcome').textContent = name;
```

## Source i sink

Source to miejsce, z którego pochodzi data kontrolowana przez atakującego.

Przykłady:

- `location.search`,
- `location.hash`,
- `location.href`,
- `document.referrer`,
- `localStorage`,
- `sessionStorage`,
- `postMessage`.

Sink to miejsce, w którym dane są używane w potencjalnie niebezpieczny sposób.

Przykłady:

- `innerHTML`,
- `outerHTML`,
- `document.write()`,
- `insertAdjacentHTML()`,
- `eval()`,
- `setTimeout(string)`,
- React `dangerouslySetInnerHTML`.

## Dlaczego kontekst ma znaczenie

Ten sam input może być bezpieczny w jednym kontekście i niebezpieczny w innym.

Dane mogą trafić do:

- HTML body,
- HTML attribute,
- JavaScript string,
- URL,
- CSS,
- DOM API.

Dlatego testowanie XSS to nie tylko sprawdzenie, czy wartość pojawia się na stronie.

Lepsze pytanie brzmi:

> Gdzie pojawia się wartość i jak przeglądarka ją zinterpretuje?

## Frontend Developer Takeaway

React pomaga, bo domyślnie escapuje wartości renderowane w JSX.

Zwykle bezpieczniejsze:

```jsx
<p>{userInput}</p>
```

Bardziej ryzykowne:

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

React nie chroni jednak automatycznie przed każdym XSS.

XSS nadal może pojawić się przez:

- `dangerouslySetInnerHTML`,
- bezpośrednią manipulację DOM,
- niebezpieczne renderowanie markdown albo rich text,
- CMS content,
- third-party components,
- unsafe URL handling,
- źle sanitizowany HTML z API.

Wzorzec do code review:

```text
user-controlled data -> unsafe rendering API
```

## Jak zapobiegać XSS

Dobre praktyki:

- używać context-aware output encoding,
- traktować input użytkownika jako tekst, nie HTML,
- używać `textContent` zamiast `innerHTML` dla zwykłego tekstu,
- sanitizować rich text/HTML, jeśli aplikacja naprawdę musi pozwalać na HTML,
- unikać `eval()`, `document.write()`, inline event handlers i unsafe DOM APIs,
- unikać React `dangerouslySetInnerHTML`, jeśli HTML nie jest zaufany albo sanitizowany,
- używać Content Security Policy jako dodatkowej warstwy, nie jedynej ochrony,
- dodawać testy regresji dla ryzykownych ścieżek renderowania.

## Review Checklist

- Czy te dane pochodzą od użytkownika albo niezaufanego źródła?
- Czy są renderowane jako tekst, czy jako HTML?
- Czy trafiają do `innerHTML`, `outerHTML`, `document.write()`, `eval()` albo `dangerouslySetInnerHTML`?
- Czy encoding pasuje do kontekstu?
- Jeśli HTML jest dozwolony, czy jest sanitizowany allowlistą?
- Czy tę treść zobaczy inny użytkownik albo admin?
- Czy CSP zmniejszyłoby impact, jeśli coś zostanie przeoczone?
- Czy istnieje test regresji dla tej ścieżki renderowania?

## Główna lekcja

XSS to problem niebezpiecznego przepływu danych do przeglądarki.

Dla frontend developera łączy się to bezpośrednio z codzienną pracą: renderowaniem danych z API, CMS, markdown, formularzy, parametrów URL i rich text.

Przeglądarka wykona kod, jeśli przypadkiem damy jej kod.

Kluczowy nawyk:

> Śledź przepływ danych od source do sink i upewnij się, że niezaufane dane są renderowane bezpiecznie.
