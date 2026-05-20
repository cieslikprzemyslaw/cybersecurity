# Ryzykowne wzorce XSS w Vue

## Cel

Vue domyślnie escapuje normalną interpolację tekstu. Ryzyko XSS rośnie, gdy developer renderuje raw HTML albo ufa dynamicznym atrybutom, takim jak URL-e.

## Bezpieczne domyślnie

```vue
<p>{{ userInput }}</p>
```

Vue traktuje to jako tekst i escapuje znaki HTML.

## Ryzykowny wzorzec: `v-html`

```vue
<div v-html="userInput"></div>
```

## Dlaczego to ryzykowne?

`v-html` renderuje raw HTML. Jeśli `userInput` pochodzi od użytkownika, CMS, markdown, API response, komentarza, profilu albo ticketu, może prowadzić do XSS.

`v-html` warto traktować podobnie jak `dangerouslySetInnerHTML` w React.

## Ryzykowny wzorzec: dynamiczne URL-e

```vue
<a :href="userControlledUrl">Open</a>
```

## Dlaczego to ryzykowne?

Jeśli URL jest kontrolowany przez użytkownika, może używać niebezpiecznego schematu, np. `javascript:`, albo prowadzić do nieoczekiwanego contentu.

## Bezpieczniejsze alternatywy

- Używaj normalnej interpolacji dla tekstu.
- Unikaj `v-html` dla user-controlled content.
- Sanitizuj rich text przed renderowaniem.
- Pozwalaj tylko na oczekiwane tagi i atrybuty.
- Waliduj URL-e i ogranicz dozwolone protokoły.

## Checklist do review

- Czy użyto `v-html`?
- Skąd pochodzi HTML?
- Czy HTML jest sanitizowany?
- Czy sanitizer używa allowlisty?
- Czy dynamiczne `href` albo `src` są kontrolowane przez użytkownika?
- Czy protokoły są ograniczone do bezpiecznych wartości, np. `https:`?
- Czy renderowany jest raw CMS/markdown content?

## Developer takeaway

```text
Vue jest bezpieczny przy normalnej interpolacji tekstu, ale v-html renderuje raw HTML i wymaga security review.
```
