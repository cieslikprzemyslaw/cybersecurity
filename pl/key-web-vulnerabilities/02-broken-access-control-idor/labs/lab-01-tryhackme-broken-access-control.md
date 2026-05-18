# Lab 01 - TryHackMe Broken Access Control

> **Platforma:** TryHackMe  
> **Temat:** Broken Access Control  
> **Status:** Completed  
> **Typ:** Notatki z laba, nie pełny walkthrough

---

## Fokus praktyki

Ten room pokazuje Broken Access Control jako problem autoryzacji.

Główny fokus to różnica między:

- authentication: kim jest użytkownik,
- authorization: do czego użytkownik ma dostęp i co może zrobić.

---

## Kluczowe pojęcie

Broken Access Control występuje wtedy, gdy aplikacja nie egzekwuje poprawnie zasad dostępu.

Użytkownik może być poprawnie zalogowany, ale nadal nie powinien móc:

- dostać się do danych innego użytkownika,
- używać funkcji admina,
- modyfikować zasobów, których nie posiada,
- wykonywać akcji spoza swojej roli.

---

## Kluczowe punkty

- Access control musi być egzekwowany po stronie serwera.
- Ograniczenia frontendowe są użyteczne dla UX, ale nie są kontrolą bezpieczeństwa.
- Problemy kontroli dostępu mogą dotyczyć zarówno odczytu danych, jak i wykonywania akcji.
- Authorization checks powinny działać dla każdego wrażliwego endpointu.
- Bezpieczny design powinien działać w modelu deny-by-default.

---

## Wniosek developerski

Przy implementacji funkcji warto pytać:

```text
Kto może uzyskać dostęp do tego zasobu?
Kto może wykonać tę akcję?
Gdzie backend to egzekwuje?
```

---

## Remediacja

Dobre praktyki:

- server-side authorization checks,
- least privilege,
- deny-by-default access control,
- centralne permission checks, jeśli to możliwe,
- automatyczne testy dla nieautoryzowanego dostępu,
- brak zaufania do wartości z klienta, takich jak `role`, `isAdmin` albo `userId`.

---

## Główna zasada

```text
Zalogowany użytkownik nie oznacza, że ma dostęp do każdego obiektu i każdej akcji.
```
