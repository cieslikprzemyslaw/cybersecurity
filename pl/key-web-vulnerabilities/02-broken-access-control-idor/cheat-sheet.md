# Broken Access Control i IDOR - Praktyczna checklista

> **Cel:** Szybka checklista do legalnych labów, autoryzowanych testów, secure code review i remediacji developerskiej.  
> **Status:** Defensive / educational. Nie jest to pełny walkthrough.

---

## Główna idea

```text
Authenticated nie znaczy authorized.
```

To, że użytkownik jest zalogowany, potwierdza tylko jego tożsamość. Nie potwierdza, że może uzyskać dostęp do konkretnego obiektu albo wykonać konkretną akcję.

---

## Typowe ryzykowne wzorce

Warto szukać wartości takich jak:

```text
id
userId
accountId
customerId
invoiceId
orderId
fileId
documentId
profileId
role
isAdmin
permission
owner
filename
download
transcript
```

Referencje do obiektów mogą znajdować się w:

- query parameters,
- ścieżkach URL,
- JSON body,
- polach formularza,
- hidden inputs,
- cookies,
- headers,
- nazwach plików,
- endpointach API,
- GraphQL variables,
- wartościach zakodowanych.

---

## Workflow testowania

### 1. Zmapuj funkcję

Pytania:

- Jaki zasób jest używany?
- Jaka akcja jest wykonywana?
- Kto powinien mieć do tego prawo?
- Czy akcja tylko odczytuje dane, czy zmienia stan?

### 2. Znajdź referencje do obiektów

Szukaj ID, nazw plików, UUID, referencji kont, dokumentów albo wartości powiązanych z użytkownikiem.

Przykład:

```http
GET /api/invoices/123
GET /account?id=123
GET /download?file=1.txt
POST /api/profile/update
```

### 3. Przetestuj inny obiekt

Zmień referencję do obiektu.

Przykład:

```text
id=123 -> id=124
/users/123 -> /users/124
1.txt -> 2.txt
invoiceA.pdf -> invoiceB.pdf
```

### 4. Użyj dwóch kont, jeśli to możliwe

Najlepszy wzorzec:

```text
Konto A tworzy albo posiada zasób.
Konto B próbuje odczytać albo zmodyfikować zasób konta A.
```

### 5. Porównaj odpowiedzi

Oczekiwane bezpieczne odpowiedzi:

- `401 Unauthorized`,
- `403 Forbidden`,
- `404 Not Found`,
- redirect do logowania, jeśli użytkownik nie jest zalogowany.

Podejrzana odpowiedź:

```text
200 OK + dane albo akcja należąca do innego użytkownika
```

---

## Pytania w Burp Repeater

- Czy można zmienić ID i nadal dostać dane?
- Czy można zmienić nazwę pliku i pobrać inny plik?
- Czy można zmienić `userId` w request body?
- Czy można zmienić `role` albo `isAdmin`?
- Czy request admina działa po odtworzeniu jako zwykły użytkownik?
- Czy frontend ukrywa akcję, którą backend nadal przyjmuje?
- Czy autoryzacja jest sprawdzana dla każdej metody i trasy?
- Czy odpowiedź wycieka dane nawet po redirect?
- Czy endpoint zachowuje się inaczej dla różnych użytkowników?

---

## Red flags w frontendzie / API

### Ufanie ID z klienta

```ts
getProfile(req.query.userId)
```

Lepszy wzorzec:

```ts
getProfileForCurrentUser(req.session.userId)
```

### Ufanie roli z requestu

```ts
if (req.body.isAdmin) {
  performAdminAction()
}
```

Lepszy wzorzec:

```ts
if (currentUser.role === "admin") {
  performAdminAction()
}
```

### Ochrona tylko we frontendzie

```tsx
{user.role === "admin" && <DeleteUserButton />}
```

To jest OK dla UX, ale nie jest kontrolą bezpieczeństwa. Backend nadal musi egzekwować autoryzację.

---

## Checklista remediacji

Backend powinien:

- brać obecnego użytkownika z sesji albo tokena,
- nie ufać `userId`, `role` ani `isAdmin` z klienta,
- sprawdzać ownership przed zwróceniem albo zmianą danych,
- sprawdzać permissions dla każdej wrażliwej akcji,
- stosować deny-by-default,
- używać centralnych helperów albo middleware do autoryzacji, jeśli to możliwe,
- pisać testy dla cross-user access,
- logować podejrzane próby dostępu,
- nie ujawniać wrażliwych danych w redirectach ani komunikatach błędów.

---

## Pomysły na testy regresji

Dla każdego wrażliwego endpointu:

```text
User A posiada Resource A.
User B próbuje odczytać Resource A.
User B próbuje zmodyfikować Resource A.
User B powinien dostać 403 albo 404.
```

Dla akcji admin-only:

```text
Admin może wykonać akcję.
Zwykły użytkownik nie może wykonać akcji.
Anonimowy użytkownik nie może wykonać akcji.
```

Dla pobierania plików/obiektów:

```text
User A pobiera własny plik.
User B nie może pobrać pliku User A.
Zmiana ID/nazwy pliku nie ujawnia pliku innego użytkownika.
```

---

## Bezpieczne odpowiedzi

| Scenariusz | Bezpieczniejsza odpowiedź |
|---|---|
| Brak logowania | `401` albo redirect do logowania |
| Zalogowany, ale brak uprawnień | `403` |
| Ukrycie istnienia obiektu | `404` |
| Poprawny autoryzowany dostęp | `200` z dozwolonymi danymi |

Unikać:

```text
200 OK + dane innego użytkownika
302 redirect + wrażliwe dane w body odpowiedzi
verbose errors ujawniające szczegóły obiektu
```

---

## OWASP / ASVS Mapping

- OWASP Top 10: Broken Access Control
- Powiązane zasady:
  - deny by default,
  - least privilege,
  - server-side authorization,
  - object-level authorization,
  - function-level authorization,
  - secure defaults.

---

## Główny wniosek developerski

```text
Klient może zażądać czegokolwiek. Serwer musi zdecydować, co jest dozwolone.
```

Kontrolki we frontendzie poprawiają UX, ale system chroni autoryzacja po stronie backendu.

## Powiązane notatki

- [Overview](overview.md)
- [Podsumowanie modułu](summary.md)
- [Laby](labs/README.md)
