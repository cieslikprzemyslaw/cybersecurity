# Lab Summary: PortSwigger — Reflected XSS into Attribute with Angle Brackets HTML-Encoded

> **Platforma:** PortSwigger Web Security Academy  
> **Temat:** XSS context / HTML attribute context  
> **Status:** Ukończone  
> **Typ notatki:** Podsumowanie nauki, nie pełny walkthrough

---

## Co testowałem

Testowałem reflected XSS, gdzie user input pojawiał się wewnątrz HTML attribute, a nie bezpośrednio między tagami HTML.

Najważniejsze pytanie:

```text
W jakim kontekście pojawia się mój input?
```

---

## Co znalazłem

Input pojawiał się w atrybucie podobnym do:

```html
<input value="USER_INPUT">
```

Angle brackets były HTML-encoded, więc stworzenie nowego taga nie było właściwą drogą.

Najważniejsza lekcja: trzeba było zrozumieć, jak wyjść z istniejącego attribute context i dodać prawdziwy event handler.

---

## Przyczyna źródłowa

Aplikacja umieszczała user-controlled input w HTML attribute bez bezpiecznego encodingu dla pełnego kontekstu atrybutu.

---

## Dlaczego kontekst miał znaczenie

Basic HTML-body payload może nie zadziałać w attribute context.

W tym labie najważniejsza idea była taka:

```text
zamknij aktualny atrybut → dodaj prawdziwy browser event handler → utrzymaj poprawne parsowanie HTML
```

To pokazało, że XSS nie polega na zapamiętaniu jednego payloadu. Chodzi o zrozumienie, jak browser parsuje końcowy HTML.

---

## Wpływ

Atakujący mógłby stworzyć request, w którym reflected input modyfikuje wygenerowany HTML attribute i powoduje wykonanie JavaScriptu w przeglądarce ofiary.

---

## Remediacja dla developerów

- Używaj context-aware output encoding dla HTML attributes.
- Bezpiecznie quote attributes.
- Unikaj wpisywania user input bezpośrednio do attributes, jeśli nie jest to potrzebne.
- Waliduj input według oczekiwanego formatu.
- Używaj bezpiecznych defaultów frameworka/template i unikaj ręcznego składania HTML.

---

## Pomysł na test regresji

Sprawdź, że quotes, angle brackets i event-handler-like input są bezpiecznie encoded, gdy trafiają do HTML attributes.

---

## Najważniejszy wniosek

```text
Payload zależy od kontekstu. HTML body context i HTML attribute context wymagają innego myślenia.
```
