# Broken Access Control i IDOR - Laby

> Krótkie podsumowania legalnych labów wykonanych dla tego tematu.  
> To są notatki praktyczne, nie pełne walkthrough ani gotowe odpowiedzi.

---

## Laby

| Platforma | Lab / Room | Fokus | Notatki |
|---|---|---|---|
| TryHackMe | Lab 01 - Broken Access Control | AuthN vs AuthZ, kategorie kontroli dostępu, server-side checks | [Podsumowanie](lab-01-tryhackme-broken-access-control.md) |
| TryHackMe | Lab 02 - IDOR | Referencje do obiektów, zmiana ID, zachowanie backendu | [Podsumowanie](lab-02-tryhackme-idor.md) |
| PortSwigger | Lab 03 - User ID controlled by request parameter | Horizontal privilege escalation przez ID kontrolowane przez użytkownika | [Podsumowanie](lab-03-portswigger-user-id-controlled-by-request-parameter.md) |
| PortSwigger | Lab 04 - Insecure direct object references | IDOR przez transkrypty rozmów i referencje do plików | [Podsumowanie](lab-04-portswigger-insecure-direct-object-references.md) |

---

## Zakres praktyki

- Szukanie referencji do obiektów kontrolowanych przez użytkownika.
- Modyfikowanie ID, nazw plików i innych referencji.
- Porównywanie odpowiedzi serwera.
- Analiza ownership checks.
- Potwierdzenie, że frontend nie jest granicą bezpieczeństwa.
- Pisanie notatek remediacyjnych z perspektywy developera.

---

## Główny wzorzec

```text
Referencja do obiektu kontrolowana przez użytkownika + brak autoryzacji po stronie serwera = Broken Access Control
```

## Wnioski z labów

- IDOR jest typem Broken Access Control.
- Ryzykowna wartość nie zawsze nazywa się `id`.
- Referencje mogą występować w query parameters, ścieżkach, request body, nazwach plików, linkach download, cookies, headers albo wartościach zakodowanych.
- Check po stronie frontendu może poprawić UX, ale nie egzekwuje autoryzacji.
- Najbardziej wiarygodny wzorzec testowania to porównanie dostępu między dwoma kontami.
- Bezpieczna implementacja powinna sprawdzać ownership albo permissions po stronie serwera przed zwróceniem albo zmianą danych.

## Powiązane notatki

- [Overview](../overview.md)
- [Podsumowanie modułu](../summary.md)
- [Praktyczna checklista](../cheat-sheet.md)

---

## Bezpieczny zakres testowania

Te notatki bazują na legalnych labach i autoryzowanych platformach treningowych.

Nie należy testować prawdziwych aplikacji bez zgody, jasnego zakresu i safe harbour.
