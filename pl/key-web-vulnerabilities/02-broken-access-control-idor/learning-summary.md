# Broken Access Control i IDOR - Learning Summary

> **Temat:** Broken Access Control, IDOR, object-level authorization  
> **Status:** Ukończone notatki modułu  
> **Laby:** TryHackMe + PortSwigger Web Security Academy

---

## Overview

Ta notatka podsumowuje najważniejsze punkty z labów dotyczących kontroli dostępu:

- TryHackMe: Broken Access Control
- TryHackMe: IDOR
- PortSwigger Web Security Academy: User ID controlled by request parameter
- PortSwigger Web Security Academy: Insecure direct object references

Główny fokus: zrozumienie, jak logika autoryzacji może zawieść, gdy backend ufa referencjom do obiektów kontrolowanym przez użytkownika.

---

## Najważniejsze wnioski

Broken Access Control jest przede wszystkim błędem autoryzacji.

Authentication potwierdza, kim jest użytkownik. Authorization decyduje, do czego ten użytkownik ma dostęp i co może zrobić.

Główna zasada:

```text
Authenticated nie znaczy authorized.
```

IDOR jest praktycznym przykładem tego problemu. Występuje wtedy, gdy aplikacja ujawnia referencję do obiektu, a backend nie sprawdza, czy obecny użytkownik ma prawo do obiektu wskazanego przez tę referencję.

---

## Kluczowe pojęcia

- IDOR jest typem Broken Access Control.
- Referencje do obiektów nie zawsze są prostymi numerycznymi ID.
- Referencje mogą występować w query parameters, ścieżkach URL, request body, cookies, headers, nazwach plików, linkach download, GraphQL variables albo wartościach zakodowanych.
- Ograniczenia tylko po stronie frontendu są dobre dla UX, ale nie są kontrolą bezpieczeństwa.
- Backend musi sprawdzać autoryzację dla każdego wrażliwego obiektu i każdej wrażliwej akcji.
- Bezpieczna kontrola dostępu powinna działać w modelu deny-by-default.

---

## Praktyczny wzorzec testowania

Przydatny workflow:

1. Zaloguj się jako zwykły użytkownik.
2. Znajdź requesty odnoszące się do użytkowników, kont, plików, zamówień, faktur albo innych obiektów.
3. Zmień referencję do obiektu.
4. Porównaj odpowiedź.
5. Potwierdź, czy backend sprawdza ownership albo permission.

Najmocniejszy wzorzec testowy wykorzystuje dwa konta:

```text
Konto A posiada zasób.
Konto B próbuje odczytać albo zmodyfikować zasób konta A.
```

Jeśli konto B może uzyskać dostęp, aplikacja prawdopodobnie ma problem Broken Access Control.

---

## Wniosek developerski

Nie należy ufać wartościom kontrolowanym przez klienta, takim jak:

```text
userId
accountId
invoiceId
fileId
role
isAdmin
filename
```

Backend powinien ustalić obecnego użytkownika z sesji albo tokena, a następnie sprawdzić, czy ten użytkownik ma prawo do żądanego obiektu albo akcji.

Ryzykowny wzorzec:

```ts
getInvoice(req.query.invoiceId)
```

Bezpieczniejszy wzorzec:

```ts
getInvoiceForUser(req.session.userId, req.query.invoiceId)
```

---

## Finalna zasada

Klient może zażądać czegokolwiek.

Serwer musi zdecydować, co jest dozwolone.
