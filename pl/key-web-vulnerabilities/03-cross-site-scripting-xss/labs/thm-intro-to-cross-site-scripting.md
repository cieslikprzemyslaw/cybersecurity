# Lab Summary: TryHackMe — Intro to Cross-site Scripting

> **Platforma:** TryHackMe  
> **Temat:** Podstawy XSS  
> **Status:** Ukończone  
> **Typ notatki:** Podsumowanie nauki, nie pełny walkthrough

---

## Co ćwiczyłem

Przerobiłem podstawowe typy XSS i to, jak dane kontrolowane przez użytkownika mogą być odbite, zapisane albo obsłużone przez JavaScript po stronie przeglądarki.

Room pomógł zbudować model myślenia dla:

- reflected XSS,
- stored XSS,
- DOM XSS,
- payloadów jako proof of concept,
- znaczenia contextu,
- tego, że problemem jest browser execution.

---

## Kluczowe pojęcia

### Reflected XSS

Input z aktualnego requestu trafia do natychmiastowej odpowiedzi w niebezpieczny sposób.

### Stored XSS

Input jest zapisany przez aplikację i później wyświetlany użytkownikom w niebezpieczny sposób.

### DOM XSS

Client-side JavaScript bierze dane kontrolowane przez atakującego i zapisuje je do DOM przez unsafe sink.

---

## Czego się nauczyłem

Najważniejsza lekcja: XSS to nie tylko zapamiętanie payloadów. Ważniejsze jest zrozumienie, gdzie input się pojawia i jak przeglądarka go interpretuje.

```text
source → context → sink/rendering → browser behaviour
```

---

## Lekcja dla developera

Developerzy powinni unikać renderowania niezaufanych danych jako HTML, chyba że jest to naprawdę potrzebne i bezpiecznie sanitizowane.

Dla zwykłego tekstu lepiej renderować jako tekst:

```js
element.textContent = userInput;
```

albo w React:

```jsx
<p>{userInput}</p>
```

---

## Najważniejszy wniosek

```text
XSS występuje wtedy, gdy niezaufane dane trafiają do kontekstu przeglądarki, gdzie mogą być potraktowane jako kod.
```
