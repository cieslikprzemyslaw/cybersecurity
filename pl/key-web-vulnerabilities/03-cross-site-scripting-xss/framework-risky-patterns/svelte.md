# Ryzykowne wzorce XSS w Svelte

## Cel

Svelte domyślnie escapuje normalne wartości. Ryzyko XSS rośnie, gdy developer używa raw HTML rendering.

## Bezpieczne domyślnie

```svelte
<p>{userInput}</p>
```

Svelte traktuje `userInput` jako tekst i escapuje znaki HTML.

## Ryzykowny wzorzec: `{@html ...}`

```svelte
{@html userInput}
```

## Dlaczego to ryzykowne?

`{@html}` renderuje raw HTML. Jeśli `userInput` pochodzi od użytkowników, CMS, markdown, API responses, komentarzy, profili albo systemów zewnętrznych, może prowadzić do XSS.

## Bezpieczniejsze alternatywy

- Używaj normalnego renderowania tekstu w Svelte.
- Unikaj raw HTML, jeśli nie ma jasnej potrzeby biznesowej.
- Sanitizuj niezaufany HTML przed renderowaniem.
- Używaj ścisłej allowlisty bezpiecznych tagów i atrybutów.
- Waliduj dynamiczne URL-e.

## Checklist do review

- Czy użyto `{@html}`?
- Skąd pochodzi HTML?
- Czy content jest user-controlled albo external-controlled?
- Czy jest sanitizacja?
- Czy event handlery i scripts są zablokowane?
- Czy unsafe URL schemes są zablokowane?
- Czy można użyć normalnego renderowania tekstu?

## Developer takeaway

```text
W Svelte {@html} oznacza raw HTML rendering i powinno odpalać security review.
```
