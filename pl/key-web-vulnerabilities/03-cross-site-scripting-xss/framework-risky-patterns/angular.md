# Ryzykowne wzorce XSS w Angular

## Cel

Angular ma wbudowane zabezpieczenia i sanitizację w wielu kontekstach template. Ryzyko XSS rośnie, gdy developer omija model bezpieczeństwa Angulara albo używa raw DOM APIs.

## Bezpieczne domyślnie

Angular template binding zwykle escapuje albo sanitizuje wartości zależnie od kontekstu.

```html
<p>{{ userInput }}</p>
```

To traktuje wartość jako tekst.

## Ryzykowne wzorce: omijanie zabezpieczeń Angulara

Sprawdzaj każde użycie:

```ts
bypassSecurityTrustHtml()
bypassSecurityTrustUrl()
bypassSecurityTrustResourceUrl()
bypassSecurityTrustScript()
bypassSecurityTrustStyle()
```

## Dlaczego to ryzykowne?

Metody `bypassSecurityTrust...` mówią Angularowi, żeby zaufał wartości, którą normalnie by chronił.

Może to być poprawne dla bardzo kontrolowanego contentu, ale jest ryzykowne, gdy wartość pochodzi od użytkowników, CMS, markdown, API, komentarzy, ticketów lub systemów zewnętrznych.

## Ryzykowny wzorzec: raw DOM APIs

```ts
elementRef.nativeElement.innerHTML = userInput;
```

albo:

```ts
document.write(userInput);
```

## Dlaczego to ryzykowne?

Bezpośrednia manipulacja DOM może ominąć zabezpieczenia template Angulara. Jeśli attacker-controlled input trafi do `innerHTML`, może stać się wykonywalnym HTML.

## Bezpieczniejsze alternatywy

- Preferuj Angular interpolation dla tekstu.
- Unikaj `bypassSecurityTrust...`, chyba że jest mocny powód.
- Sanitizuj niezaufany HTML przed renderowaniem.
- Używaj ścisłych allowlist dla rich text.
- Waliduj URL-e i dozwolone protokoły.

## Checklist do review

- Czy użyto `bypassSecurityTrust...`?
- Dlaczego jest potrzebne?
- Czy wartość jest w pełni zaufana?
- Czy pochodzi od użytkownika, CMS albo z zewnątrz?
- Czy użyto raw DOM manipulation?
- Czy `innerHTML` jest ustawiany bezpośrednio?
- Czy URL-e są walidowane?
- Czy content jest sanitizowany przez ścisłą allowlistę?

## Developer takeaway

```text
Jeśli kod Angular mówi “bypass security”, traktuj to jako punkt security review.
```

Angular pomaga domyślnie, ale bypass APIs i direct DOM manipulation mogą przywrócić ryzyko XSS.
