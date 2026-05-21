# Lab Summary: PortSwigger — Stored XSS into HTML Context with Nothing Encoded

> **Platforma:** PortSwigger Web Security Academy  
> **Temat:** Stored XSS  
> **Status:** Ukończone  
> **Typ notatki:** Podsumowanie nauki, nie pełny walkthrough

---

## Co testowałem

Testowałem funkcję, w której user-submitted content był zapisywany przez aplikację i później wyświetlany na stronie.

Najważniejsze pytanie:

```text
Czy dane wysłane przez jednego użytkownika są zapisane i później renderowane niebezpiecznie dla innych użytkowników?
```

---

## Co znalazłem

Aplikacja zapisywała niezaufany input i później renderowała go w HTML context bez safe output encoding albo sanitization.

---

## Przyczyna źródłowa

Aplikacja ufała stored user-generated content i renderowała go jako aktywny HTML.

```text
stored input → later response → browser execution
```

---

## Wpływ

Stored XSS może być groźniejszy niż reflected XSS, bo ofiara nie musi kliknąć crafted link. Wystarczy, że otworzy stronę, gdzie payload został zapisany.

Możliwy impact:

- atak na użytkowników czytających komentarze albo wiadomości,
- atak na administratorów przeglądających submitted content,
- wykonywanie akcji jako ofiara,
- modyfikacja tego, co użytkownicy widzą w aplikacji.

---

## Remediacja dla developerów

- Encode output przy renderowaniu stored user content.
- Sanitizuj tylko wtedy, gdy aplikacja musi wspierać ograniczony HTML/rich text.
- Używaj strict allowlist dla allowed tags i attributes.
- Nie renderuj komentarzy, profili ani support messages jako raw HTML.
- Dokładnie sprawdzaj admin dashboards, bo stored XSS często dotyka privileged users.

---

## Pomysł na test regresji

Utwórz content zawierający HTML-like input i sprawdź, czy jest wyświetlany jako tekst, a nie wykonywany, zarówno dla normalnego użytkownika, jak i admina.

---

## Najważniejszy wniosek

```text
Stored XSS jest groźne, bo payload staje się częścią danych aplikacji i może dotknąć przyszłych użytkowników.
```
