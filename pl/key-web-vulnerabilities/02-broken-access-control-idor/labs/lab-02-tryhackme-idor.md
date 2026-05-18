# Lab 02 - TryHackMe IDOR

> **Platforma:** TryHackMe  
> **Temat:** Insecure Direct Object Reference  
> **Status:** Completed  
> **Typ:** Notatki z laba, nie pełny walkthrough

---

## Fokus praktyki

Ten room skupia się na zmianie referencji do obiektów i obserwacji, czy backend poprawnie egzekwuje autoryzację.

Przykłady referencji do obiektów:

- user IDs,
- account IDs,
- document IDs,
- nazwy plików,
- API object identifiers.

---

## Kluczowe pojęcie

IDOR występuje wtedy, gdy aplikacja ujawnia referencję do obiektu i nie sprawdza, czy obecny użytkownik ma prawo do tego obiektu.

Przykład:

```http
GET /resource?id=123
```

Jeśli zmiana ID zwraca zasób innego użytkownika, backendowi brakuje ownership albo permission check.

---

## Kluczowe punkty

- IDOR jest typem Broken Access Control.
- IDOR nie dotyczy tylko numerycznych ID.
- Referencje mogą występować w URL, ścieżkach, request body, headers, cookies albo wartościach zakodowanych.
- Problemem nie jest to, że użytkownik może zmodyfikować request.
- Problemem jest to, że backend ufa zmodyfikowanej referencji.

---

## Wniosek developerski

Backend nie powinien polegać wyłącznie na referencji do obiektu.

Powinien sprawdzać:

```text
żądany obiekt + obecny zalogowany użytkownik + permission/ownership
```

---

## Remediacja

Bezpieczniejsze wzorce:

- brać obecnego użytkownika z sesji/tokena,
- sprawdzać ownership przed zwróceniem danych,
- nie ufać `userId` ani `accountId` z klienta,
- stosować deny-by-default,
- testować dostęp z użyciem dwóch kont.

---

## Główna zasada

```text
IDOR dotyczy braku autoryzacji dla obiektu wskazanego przez ID.
```
