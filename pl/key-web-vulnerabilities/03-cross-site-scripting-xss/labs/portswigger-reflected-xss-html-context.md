# Lab Summary: PortSwigger — Reflected XSS into HTML Context with Nothing Encoded

> **Platforma:** PortSwigger Web Security Academy  
> **Temat:** Reflected XSS  
> **Status:** Ukończone  
> **Typ notatki:** Podsumowanie nauki, nie pełny walkthrough

---

## Co testowałem

Testowałem funkcję search, w której input użytkownika był odbijany z powrotem na stronie.

Najważniejsze pytanie:

```text
Czy mój input pojawia się w natychmiastowej odpowiedzi HTML?
```

---

## Co znalazłem

Aplikacja odbijała search input bezpośrednio w HTML response bez bezpiecznego encode’owania.

To tworzyło reflected XSS, ponieważ przeglądarka mogła zinterpretować attacker-controlled input jako aktywną treść.

---

## Root Cause

Aplikacja renderowała dane z requestu w HTML context bez context-aware output encoding.

```text
request parameter → HTML response → browser interprets as markup/code
```

---

## Impact

Atakujący mógłby stworzyć link powodujący wykonanie JavaScriptu w przeglądarce ofiary.

W zależności od aplikacji i uprawnień ofiary może to prowadzić do:

- akcji wykonanych jako ofiara,
- odczytu danych widocznych dla ofiary,
- manipulacji UI,
- atakowania użytkowników z wyższymi uprawnieniami.

---

## Developer Remediation

- Encode output przed renderowaniem user input w HTML.
- Traktuj query parameters jako niezaufane.
- Używaj bezpiecznych defaultów template/frameworka.
- Unikaj ręcznego doklejania user input do HTML.
- Dodaj testy pokazujące, że search terms są wyświetlane jako tekst, a nie wykonywane jako kod.

---

## Regression Test Idea

Wyślij payload-like input do search field i sprawdź, czy pojawia się jako escaped text w response.

---

## Najważniejszy wniosek

```text
Reflected XSS polega na niebezpiecznym natychmiastowym odbiciu request input w response.
```
