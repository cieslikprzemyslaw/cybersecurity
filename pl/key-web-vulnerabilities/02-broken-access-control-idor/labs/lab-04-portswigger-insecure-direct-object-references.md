# Lab 04 - PortSwigger: Insecure Direct Object References

> **Platforma:** PortSwigger Web Security Academy  
> **Temat:** Access Control / IDOR  
> **Status:** Completed  
> **Typ:** Notatki z laba, nie pełny walkthrough

---

## Fokus praktyki

Ten lab pokazuje, że IDOR nie zawsze jest prostym parametrem `?id=123`.

Podatna referencja do obiektu dotyczyła plików transkryptów rozmów, które były pobierane przez przewidywalne referencje do plików.

---

## Wzorzec podatności

Aplikacja ujawniała bezpośrednie referencje do plików transkryptów.

Jeśli użytkownik może zmienić wskazany plik i uzyskać dostęp do transkryptu innego użytkownika, backend nie egzekwuje kontroli dostępu dla tego pliku.

---

## Co było testowane

- Gdzie znajdują się referencje do plików transkryptów.
- Czy nazwy plików albo URL są przewidywalne.
- Czy zmiana referencji do pliku zwraca inny transkrypt.
- Czy backend sprawdza, że obecny użytkownik ma prawo do danego transkryptu.

---

## Root Cause

Backend pozwalał na bezpośredni dostęp do plików transkryptów bez sprawdzenia ownership albo permission.

---

## Impact

Atakujący mógłby uzyskać dostęp do prywatnego transkryptu rozmowy innego użytkownika.

W prawdziwych aplikacjach podobne problemy mogą ujawniać:

- support tickets,
- chat logs,
- faktury,
- przesłane pliki,
- dokumenty wewnętrzne,
- prywatne dane użytkowników.

---

## Remediacja

Backend powinien:

- unikać ujawniania przewidywalnych bezpośrednich referencji do plików,
- mapować dostęp do plików przez warstwę autoryzacji,
- sprawdzać, czy obecny użytkownik posiada plik albo ma prawo do dostępu,
- przechowywać pliki poza publicznie dostępnymi ścieżkami, jeśli to właściwe,
- zwracać `403` albo `404`, gdy dostęp nie jest dozwolony.

---

## Główna zasada

```text
IDOR może wystąpić przez pliki i URL, nie tylko przez numeryczne ID.
```
