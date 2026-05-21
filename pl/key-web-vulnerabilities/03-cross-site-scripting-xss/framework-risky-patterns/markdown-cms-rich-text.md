# Ryzyka XSS w Markdown, CMS i Rich Text Rendering

## Cel

Markdown, pola CMS i rich text editors często produkują HTML. Ryzyko XSS pojawia się, gdy niezaufany content jest konwertowany do HTML i renderowany bez bezpiecznej sanitizacji.

## Typowe scenariusze

```text
komentarze blogowe
CMS rich text fields
support tickets
profile użytkowników
markdown descriptions
opisy produktów
strony dokumentacji
notatki admina
```

## Ryzykowny wzorzec

```tsx
<div dangerouslySetInnerHTML={{ __html: htmlFromMarkdownOrCms }} />
```

albo:

```js
content.innerHTML = htmlFromCms;
```

## Dlaczego to ryzykowne?

Content z markdown, CMS albo rich text może zawierać:

```text
<script> tags
event handlery, np. onerror albo onclick
javascript: URLs
iframes
SVG z aktywnym contentem
niebezpieczne atrybuty
nieoczekiwany HTML
```

Jeśli zostanie wyrenderowany bezpośrednio, może stać się stored XSS.

## Bezpieczniejsze podejście

- Zdecyduj, czy HTML naprawdę jest potrzebny.
- Jeśli nie, renderuj jako plain text.
- Jeśli rich text jest potrzebny, sanitizuj przed renderowaniem.
- Używaj ścisłej allowlisty tagów i atrybutów.
- Usuwaj event handlery.
- Blokuj `javascript:` URLs.
- Ogranicz iframes, SVG, scripts i niebezpieczne atrybuty.
- Rozważ serwowanie user-uploaded content z osobnej domeny/origin.
- Dodaj CSP jako defence-in-depth.

## Uwaga na Markdown

Markdown nie jest automatycznie bezpieczny, jeśli raw HTML jest dozwolony.

Sprawdzaj konfigurację markdown:

```text
Czy raw HTML jest włączony?
Czy linki są sanitizowane?
Czy obrazy są ograniczone?
Czy dangerous protocols są blokowane?
Czy output jest sanitizowany po konwersji?
```

## Uwaga na CMS

CMS content może być zaufany redakcyjnie, ale nadal jest contentem, który może stać się wykonywalny w przeglądarce.

To szczególnie ważne, gdy:

```text
jest wielu editorów
content pochodzi z integracji
content jest importowany
pola contentu wspierają raw HTML
content jest renderowany w różnych kontekstach
```

## Checklist do review

- Czy markdown jest konwertowany do HTML?
- Czy raw HTML jest dozwolony?
- Czy CMS rich text jest renderowany jako HTML?
- Czy sanitizacja jest zastosowana po konwersji?
- Czy sanitizer ma ścisłą allowlistę?
- Czy event handlery są usunięte?
- Czy dangerous URL schemes są blokowane?
- Czy SVG jest dozwolone?
- Czy content jest pokazywany innym użytkownikom?
- Czy CSP jest skonfigurowane jako defence-in-depth?

## Lekcja dla developera

```text
Rich text to nadal user/content-controlled input. Jeśli staje się HTML, wymaga sanitizacji i context-aware rendering.
```
