# Cross-Site Scripting (XSS)

To trzeci temat w serii Key Web Vulnerabilities.

## Start

1. [Overview](./overview.md)
2. [Cheat Sheet](./cheat-sheet.md)
3. [Labs](./labs/README.md)

## Zakres tematu

Ten moduł obejmuje:

- reflected XSS,
- stored XSS,
- DOM XSS,
- sources and sinks,
- HTML body context,
- HTML attribute context,
- output encoding,
- sanitization,
- bezpieczne renderowanie we frontendzie,
- ryzyka XSS w React,
- remediation z perspektywy developera.

## Przerobione laby

| Platforma | Lab / temat | Czego dotyczył |
|---|---|---|
| TryHackMe | Intro to Cross-site Scripting | Podstawy XSS i myślenie o payloadach |
| PortSwigger | Reflected XSS into HTML context with nothing encoded | Input odbity bezpośrednio w HTML |
| PortSwigger | Stored XSS into HTML context with nothing encoded | Input zapisany i wyrenderowany później |
| PortSwigger | DOM XSS in `document.write` sink using `location.search` source | Flow source → sink w JavaScript po stronie przeglądarki |
| PortSwigger | Reflected XSS into attribute with angle brackets HTML-encoded | Attribute context i event handler injection |

## Najważniejsza myśl

XSS nie polega tylko na tym, żeby odpalić `alert(1)`. Prawdziwa lekcja to zrozumienie, jak niezaufane dane trafiają do kontekstu, w którym przeglądarka może potraktować je jako kod.

```text
user-controlled input → unsafe rendering context → browser treats data as code
```

## Najważniejszy wniosek dla frontend developera

Najważniejszy model myślenia:

```text
Skąd pochodzą dane?
Gdzie trafiają?
Czy są renderowane jako tekst, czy interpretowane jako HTML/JavaScript?
```

Bezpieczne renderowanie nie jest tylko problemem backendu. Frontend może wprowadzić XSS przez unsafe DOM APIs, raw HTML rendering, markdown/rich text, CMS content albo źle przejrzane biblioteki.
