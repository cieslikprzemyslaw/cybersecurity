# Ryzykowne wzorce XSS według frameworka i kontekstu renderowania

> **Ścieżka nauki:** Frontend Engineer → Application Security
> **Temat:** Cross-Site Scripting, ryzykowne wzorce frameworkowe, bezpieczne renderowanie
> **Status:** Notatki do code review, nie kolekcja payloadów

## Cel

Ten folder zbiera typowe wzorce podatne na XSS w frontend frameworkach, plain JavaScript, PHP, server-side templates, markdown, CMS i rich text.

Celem nie jest zapamiętanie payloadów. Celem jest zrozumienie, gdzie XSS pojawia się w realnym kodzie i jak bezpiecznie go reviewować.

## Główna myśl

Nowoczesne frameworki często są bezpieczniejsze domyślnie, jeśli developer używa normalnego renderowania tekstu.

Ryzyko XSS rośnie, gdy developer omija te zabezpieczenia i renderuje niezaufane dane jako raw HTML, JavaScript, URL albo DOM content.

```text
Dane kontrolowane przez użytkownika + niebezpieczny kontekst renderowania = możliwy XSS
```

## Pliki

| Plik | Zakres |
|---|---|
| `react.md` | React, JSX escaping, `dangerouslySetInnerHTML`, ryzykowne URL/rendering patterns |
| `angular.md` | Angular sanitization, `bypassSecurityTrust*`, raw DOM usage |
| `vue.md` | Vue interpolation, `v-html`, dynamiczne URL-e |
| `svelte.md` | Svelte escaping, `{@html}` |
| `plain-javascript-dom.md` | `innerHTML`, `document.write`, `eval`, niebezpieczne DOM sinks |
| `php.md` | PHP output escaping, `htmlspecialchars`, HTML/attribute/JS contexts |
| `server-side-templates.md` | Template engines: Twig, Jinja, EJS, Handlebars, Razor, Thymeleaf |
| `markdown-cms-rich-text.md` | Markdown, CMS rich text, HTML sanitization, allowlisty |

## Pytania do code review

Podczas review kodu pytaj:

- Czy dane są kontrolowane przez użytkownika?
- Czy są renderowane jako tekst, czy jako raw HTML?
- Czy ochrona frameworka jest omijana?
- Jaki jest kontekst outputu: HTML body, HTML attribute, JavaScript, URL, CSS czy DOM?
- Czy potrzebna jest sanitizacja?
- Czy output encoding jest dopasowany do kontekstu?
- Czy możliwe są `javascript:` URLs, event handlery albo niebezpieczne tagi?
- Czy CSP jest tylko defence-in-depth, a nie jedyną ochroną?

## Developer takeaway

Frameworki pomagają, ale nie zastępują bezpiecznych decyzji przy renderowaniu.

```text
Safe defaults chronią normalne renderowanie.
Raw HTML features wymagają security review.
```
