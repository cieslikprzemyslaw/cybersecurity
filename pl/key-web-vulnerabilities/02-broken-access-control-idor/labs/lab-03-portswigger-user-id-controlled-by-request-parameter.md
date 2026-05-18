# Lab 03 - PortSwigger: User ID Controlled by Request Parameter

> **Platforma:** PortSwigger Web Security Academy  
> **Temat:** Access Control / IDOR  
> **Status:** Completed  
> **Typ:** Notatki z laba, nie pełny walkthrough

---

## Fokus praktyki

Ten lab pokazuje klasyczny horizontal privilege escalation.

Aplikacja używa parametru kontrolowanego przez użytkownika, aby zdecydować, dane którego konta zwrócić.

---

## Wzorzec podatności

Request zawiera identyfikator użytkownika, na przykład:

```http
GET /my-account?id=wiener
```

Jeśli backend przyjmuje zmieniony identyfikator i zwraca dane innego użytkownika, kontrola dostępu jest błędna.

---

## Co było testowane

- Który parametr kontroluje stronę konta użytkownika.
- Czy zmiana identyfikatora zmienia zwrócone dane konta.
- Czy backend sprawdza, że żądane konto należy do zalogowanego użytkownika.
- Jak odpowiedź zmienia się po modyfikacji parametru.

---

## Root Cause

Backend zaufał identyfikatorowi kontrolowanemu przez użytkownika bez sprawdzenia, czy zalogowany użytkownik ma prawo do żądanego konta.

---

## Impact

Atakujący mógłby uzyskać dostęp do danych konta albo wrażliwych informacji innego użytkownika.

W prawdziwych aplikacjach podobne problemy mogą ujawniać:

- dane osobowe,
- API keys,
- faktury,
- zamówienia,
- prywatne wiadomości,
- ustawienia konta.

---

## Remediacja

Backend powinien:

- identyfikować obecnego użytkownika z sesji albo tokena,
- ignorować client-controlled user IDs tam, gdzie to możliwe,
- sprawdzać ownership przed zwróceniem danych,
- zwracać `403` albo `404` przy braku uprawnień.

Ryzykowny wzorzec:

```ts
getAccount(req.query.id)
```

Bezpieczniejszy wzorzec:

```ts
getAccountForUser(req.session.userId)
```

---

## Główna zasada

```text
Zmiana user ID nie powinna dawać dostępu do konta innego użytkownika.
```
