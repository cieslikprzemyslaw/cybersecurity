# Lab Summary: PortSwigger — DOM XSS in document.write Sink Using location.search Source

> **Platforma:** PortSwigger Web Security Academy  
> **Temat:** DOM XSS  
> **Status:** Ukończone  
> **Typ notatki:** Podsumowanie nauki, nie pełny walkthrough

---

## Co testowałem

Testowałem DOM-based XSS pattern, gdzie JavaScript po stronie przeglądarki czyta dane z URL i zapisuje je do strony.

Najważniejsze pytanie:

```text
Czy attacker-controlled browser data może trafić do unsafe DOM sink?
```

---

## Source and Sink

Source:

```js
location.search
```

Sink:

```js
document.write()
```

---

## Co znalazłem

Vulnerable flow był po stronie przeglądarki, a nie typowo po stronie serwera.

Sama odpowiedź serwera nie tłumaczyła całej podatności. Problem pojawiał się, gdy JavaScript przetwarzał dane z URL i pisał je do DOM w niebezpieczny sposób.

---

## Przyczyna źródłowa

Client-side JavaScript używał danych kontrolowanych przez atakującego i przekazywał je do dangerous sink.

```text
location.search → JavaScript processing → document.write() → DOM execution
```

---

## Wpływ

Atakujący mógłby stworzyć URL powodujący wykonanie JavaScriptu w przeglądarce ofiary po przetworzeniu parametru przez podatną stronę.

---

## Remediacja dla developerów

- Unikaj `document.write()` dla danych kontrolowanych przez użytkownika.
- Unikaj unsafe DOM sinks takich jak `innerHTML`, `outerHTML`, `insertAdjacentHTML` dla untrusted input.
- Używaj `textContent` dla zwykłego tekstu.
- Waliduj i encode dane przed renderowaniem.
- Przeglądaj source-to-sink flows podczas frontend code review.

---

## Pomysł na test regresji

Dodaj test albo review rule sprawdzający, że URL parameters nie są zapisywane do DOM przez unsafe sinks.

---

## Najważniejszy wniosek

```text
DOM XSS polega na source-to-sink flow w JavaScript po stronie przeglądarki.
```
