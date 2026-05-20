# Ryzykowne wzorce XSS w React

## Cel

React jest zwykle bezpieczny domyślnie, gdy renderujemy wartości przez normalny JSX. Ryzyko rośnie, gdy developer omija escaping JSX albo wstrzykuje raw HTML.

## Bezpieczne domyślnie

React escapuje wartości renderowane w JSX.

```tsx
const userInput = "<img src=x onerror=alert(1)>";

return <p>{userInput}</p>;
```

W tym przypadku React traktuje `userInput` jako tekst, a nie jako wykonywalny HTML.

## Ryzykowny wzorzec: `dangerouslySetInnerHTML`

```tsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

## Dlaczego to ryzykowne?

`dangerouslySetInnerHTML` renderuje raw HTML do DOM. Jeśli `userInput` pochodzi od użytkownika, CMS, markdown, API response, support ticket, komentarza albo profilu, może prowadzić do XSS.

Sama nazwa ostrzega developera, że omija normalny escaping Reacta.

## Bezpieczniejsze alternatywy

Preferuj normalne renderowanie JSX:

```tsx
<p>{userInput}</p>
```

Jeśli raw HTML jest naprawdę potrzebny:

- sanitizuj HTML przed renderowaniem,
- używaj ścisłej allowlisty tagów i atrybutów,
- blokuj skrypty, event handlery, niebezpieczne URL-e i nieoczekiwane atrybuty,
- unikaj renderowania user-controlled HTML, jeśli nie ma wyraźnej potrzeby biznesowej.

## Inne punkty review w React

Sprawdzaj szczególnie:

```text
dangerouslySetInnerHTML
html-react-parser
markdown renderowany jako HTML
CMS rich text renderowany jako HTML
manualna manipulacja DOM przez innerHTML
user-controlled href lub src
javascript: URLs
third-party widgets
```

## Uwaga na URL-e i linki

Nawet jeśli React escapuje tekst, developer musi walidować URL-e.

Ryzykowny pomysł:

```tsx
<a href={userControlledUrl}>Open</a>
```

Jeśli `userControlledUrl` może być `javascript:...`, może to być ryzykowne zależnie od przeglądarki i sposobu renderowania.

Bezpieczniej:

- pozwalaj tylko na oczekiwane protokoły, np. `https:`,
- waliduj URL-e po stronie serwera i klienta, jeśli ma to sens,
- nie ufaj bezpośrednio URL-om od użytkownika.

## Checklist do review

- Czy używany jest raw HTML rendering?
- Skąd pochodzi HTML?
- Czy content jest zaufany?
- Czy użyty jest sanitizer?
- Czy sanitizer ma ścisłą allowlistę?
- Czy event handlery są zablokowane?
- Czy `javascript:` URLs są zablokowane?
- Czy renderowany jest content z CMS albo markdown?
- Czy można użyć zwykłego JSX?

## Developer takeaway

```text
React pomaga ograniczać XSS domyślnie, ale tylko jeśli dane zostają w normalnym JSX rendering.
```

Raw HTML rendering powinien zawsze odpalać security review.
