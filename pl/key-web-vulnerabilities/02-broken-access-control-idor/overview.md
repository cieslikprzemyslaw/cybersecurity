# Broken Access Control i IDOR - Overview

> **Temat:** Broken Access Control, IDOR, referencje do obiektów, autoryzacja po stronie serwera  
> **Status:** Notatki referencyjne, nie pełny walkthrough  
> **Źródła:** TryHackMe + PortSwigger Web Security Academy + OWASP

---

## TL;DR

**Broken Access Control występuje wtedy, gdy aplikacja nie egzekwuje poprawnie tego, do czego zalogowany użytkownik ma dostęp albo co może zrobić.**

**IDOR nie polega tylko na zgadywaniu ID.**  
Prawdziwy problem polega na tym, że backend nie sprawdza, czy obecny użytkownik ma prawo dostać się do obiektu wskazanego przez daną referencję.

```text
Zalogowany użytkownik + zmieniona referencja do obiektu + brak autoryzacji po stronie serwera = IDOR
```

---

## Dlaczego to ma znaczenie

Authentication i authorization to dwa różne mechanizmy kontroli.

**Authentication / AuthN** odpowiada na pytanie:

> Kim jesteś?

**Authorization / AuthZ** odpowiada na pytanie:

> Do czego masz dostęp i co wolno Ci zrobić?

Użytkownik może być poprawnie zalogowany, ale nadal nie powinien mieć dostępu do profilu, faktury, pliku, rozmowy, zamówienia, akcji admina albo ustawień konta innego użytkownika.

Broken Access Control jest głównie problemem **authorization**.

---

## Czym jest Broken Access Control?

Broken Access Control oznacza, że aplikacja nie egzekwuje poprawnie zasad dostępu po stronie serwera.

Przykłady:

- zwykły użytkownik może użyć funkcji admina,
- użytkownik A może zobaczyć albo zmodyfikować dane użytkownika B,
- użytkownik może pobrać plik należący do kogoś innego,
- zmiana wartości w requeście daje dostęp do innego obiektu,
- frontend ukrywa przycisk, ale backend nadal przyjmuje akcję,
- endpoint API ufa wartościom `userId`, `role` albo `isAdmin` z requestu.

Kluczowy problem zwykle nie brzmi: czy użytkownik jest zalogowany.  
Kluczowy problem brzmi: czy ten użytkownik ma prawo do **tego konkretnego obiektu** albo **tej konkretnej akcji**.

---

## Czym jest IDOR?

**IDOR** oznacza **Insecure Direct Object Reference**.

Występuje wtedy, gdy aplikacja ujawnia bezpośrednią referencję do obiektu i nie sprawdza, czy obecny użytkownik ma prawo do tego obiektu.

Przykład:

```http
GET /account?id=123
```

Zmienione na:

```http
GET /account?id=124
```

Jeśli serwer zwraca dane innego użytkownika, backend nie sprawdza poprawnie ownership albo permission.

Problemem nie jest to, że użytkownik zmienił ID. Użytkownik zawsze może modyfikować requesty.  
Problemem jest to, że backend zaufał referencji do obiektu bez sprawdzenia autoryzacji.

---

## Referencje do obiektów to nie tylko `?id=123`

Referencje do obiektów mogą pojawić się w wielu miejscach:

- query parameters, na przykład `?id=123`,
- ścieżki URL, na przykład `/users/123`,
- request body, na przykład `{ "userId": 123 }`,
- ukryte pola formularza,
- cookies,
- headers,
- nazwy plików, na przykład `1.txt` albo `invoice.pdf`,
- linki do pobierania,
- endpointy API,
- GraphQL variables,
- UUID,
- wartości zakodowane, na przykład Base64.

Ważny wniosek z labów: IDOR dotyczy **dostępu do obiektu**, a nie tylko numerycznego ID.

W jednym z labów PortSwigger podatna referencja dotyczyła pliku transkryptu rozmowy, a nie klasycznego parametru `id`.

---

## Typy problemów kontroli dostępu

### Horizontal Privilege Escalation

Użytkownik uzyskuje dostęp do danych albo akcji należących do innego użytkownika na tym samym poziomie uprawnień.

Przykład:

```text
Użytkownik A czyta profil użytkownika B.
```

### Vertical Privilege Escalation

Użytkownik z niższymi uprawnieniami wykonuje akcję zarezerwowaną dla użytkownika z wyższymi uprawnieniami.

Przykład:

```text
Zwykły użytkownik uzyskuje dostęp do funkcji admina.
```

### Context-Dependent Access Control

Akcja może być dozwolona w jednym kontekście, ale niedozwolona w innym.

Przykład:

```text
Użytkownik może edytować własny profil, ale nie profil innego użytkownika.
```

---

## Zakres praktyki

Praktyka dla tego tematu skupiała się na:

- podstawach kontroli dostępu i różnicy AuthN vs AuthZ,
- zmianie referencji do obiektów i obserwacji zachowania backendu,
- klasycznym IDOR przez parametry kontrolowane przez użytkownika,
- IDOR przez referencje do plików i transkryptów, nie tylko `?id=123`.

---

## Podejście do testowania

Prosty workflow:

1. Zaloguj się jako zwykły użytkownik.
2. Znajdź requesty odnoszące się do obiektów albo akcji.
3. Wyślij request do Burp Repeater.
4. Zmień referencję do obiektu albo parametr akcji.
5. Porównaj odpowiedź.
6. Sprawdź, czy serwer zwraca cudze dane albo poprawnie odmawia dostępu.

Przydatne pytanie:

> Czy backend sprawdza, że ten zasób należy do obecnego użytkownika?

Mocny wzorzec testowy to dwa konta:

```text
Konto A tworzy zasób.
Konto B próbuje odczytać albo zmodyfikować zasób konta A.
```

Jeśli konto B może uzyskać dostęp, to mocny sygnał Broken Access Control.

---

## Oczekiwane bezpieczne zachowanie

Jeśli użytkownik nie ma prawa do zasobu, serwer nie powinien zwracać danych.

| Sytuacja | Oczekiwana odpowiedź |
|---|---|
| Użytkownik nie jest zalogowany | `401 Unauthorized` albo redirect do logowania |
| Użytkownik jest zalogowany, ale nie ma uprawnień | `403 Forbidden` |
| Aplikacja nie chce zdradzać istnienia zasobu | `404 Not Found` |

Niebezpieczne zachowanie:

```text
200 OK + dane innego użytkownika
```

---

## Remediacja developerska

Dobre praktyki backendowe:

- nie ufać `userId`, `accountId`, `fileId`, `invoiceId`, `role` ani `isAdmin` z requestu,
- brać obecnego użytkownika z sesji albo tokena,
- sprawdzać ownership przed zwróceniem danych,
- sprawdzać role i permissions przy każdej wrażliwej akcji,
- stosować deny-by-default,
- stosować zasadę least privilege,
- dodawać automatyczne testy kontroli dostępu,
- logować podejrzane próby dostępu bez ujawniania wrażliwych danych.

Ryzykowny wzorzec:

```ts
getInvoice(req.query.invoiceId)
```

Bezpieczniejszy wzorzec:

```ts
getInvoiceForUser(req.session.userId, req.query.invoiceId)
```

Ważna różnica: backend sprawdza zarówno:

1. jaki obiekt został zażądany,
2. czy obecny użytkownik ma prawo do tego obiektu.

---

## OWASP / AppSec Mapping

Powiązana kategoria:

- **OWASP Top 10: Broken Access Control**

Powiązane zasady:

- never trust user input,
- principle of least privilege,
- server-side authorization,
- deny by default,
- defense in depth,
- verify access per object and per action.

---

## Główna zasada

Frontend nie jest granicą bezpieczeństwa.

Aplikacja React może ukryć przyciski, zablokować inputy albo usunąć linki z UI, ale użytkownik nadal może wysłać zmodyfikowany request przez Burp, Postman, curl, DevTools albo skrypt.

Backend zawsze musi odpowiedzieć:

> Czy ten zalogowany użytkownik ma prawo dostać się do dokładnie tego obiektu i wykonać dokładnie tę akcję?

To jest sedno Broken Access Control i IDOR.
