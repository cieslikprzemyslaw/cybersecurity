# Broken Access Control i IDOR - notatki AppSec

> Ścieżka nauki: Frontend Engineer -> Application Security  
> Temat: Broken Access Control, IDOR, referencje do obiektów, server-side authorization  
> Status: notatki edukacyjne, nie pełny walkthrough labów

## TL;DR

IDOR nie polega tylko na zgadywaniu ID.

Prawdziwy problem polega na tym, że backend nie sprawdza, czy zalogowany użytkownik ma prawo dostać się do obiektu wskazanego przez to ID.

```text
Zalogowany użytkownik + zmieniona referencja do obiektu + brak server-side authorization = IDOR
```

## Kluczowe pojęcia

Authentication pyta:

> Kim jesteś?

Authorization pyta:

> Do czego masz dostęp i co wolno Ci zrobić?

IDOR i Broken Access Control są głównie problemami authorization. Użytkownik może być poprawnie zalogowany, ale nadal nie powinien mieć dostępu do danych, plików, faktur, rozmów albo ustawień konta innego użytkownika.

## Czym jest Broken Access Control?

Broken Access Control występuje, gdy aplikacja nie egzekwuje poprawnie zasad dostępu po stronie serwera.

Przykłady:

- zwykły użytkownik może użyć funkcji admina,
- użytkownik A może zobaczyć profil użytkownika B,
- użytkownik może pobrać cudzy plik,
- zmiana ID w URL zwraca cudze dane,
- frontend ukrywa przycisk, ale backend nadal przyjmuje request.

## Czym jest IDOR?

IDOR oznacza Insecure Direct Object Reference.

Występuje wtedy, gdy aplikacja ujawnia bezpośrednią referencję do obiektu i nie sprawdza, czy obecny użytkownik ma prawo do tego obiektu.

Przykład:

```http
GET /account?id=123
```

Zmienione na:

```http
GET /account?id=124
```

Jeśli serwer zwraca dane innego użytkownika, to jest problem IDOR.

## Referencje do obiektów to nie tylko `?id=123`

Referencje mogą znajdować się w wielu miejscach:

- query parameters: `?id=123`,
- ścieżki URL: `/users/123`,
- request body: `{ "userId": 123 }`,
- hidden form fields,
- cookies,
- headers,
- nazwy plików,
- linki download,
- endpointy API,
- GraphQL variables,
- wartości zakodowane, np. Base64,
- UUID.

Ważna lekcja: IDOR dotyczy dostępu do obiektu, a nie tylko numerycznego ID w URL.

## Jak myślę o testowaniu IDOR

Prosty workflow:

1. Zaloguj się jako zwykły użytkownik.
2. Znajdź requesty, które odnoszą się do obiektów.
3. Wyślij request do Burp Repeater.
4. Zmień referencję do obiektu.
5. Porównaj response.
6. Sprawdź, czy serwer zwraca cudze dane albo poprawnie odmawia dostępu.

Mocny wzorzec testowy to dwa konta:

```text
Konto A tworzy zasób.
Konto B próbuje odczytać albo zmodyfikować zasób konta A.
```

Jeśli konto B może to zrobić, to mocny sygnał Broken Access Control.

## Oczekiwane bezpieczne zachowanie

Jeśli użytkownik nie ma prawa do zasobu, serwer nie powinien zwracać danych.

Możliwe bezpieczne odpowiedzi:

| Sytuacja | Oczekiwany response |
|---|---|
| Użytkownik nie jest zalogowany | `401 Unauthorized` albo redirect do logowania |
| Użytkownik jest zalogowany, ale nie ma uprawnień | `403 Forbidden` |
| Aplikacja nie chce zdradzać istnienia zasobu | `404 Not Found` |

Niebezpieczny response:

```text
200 OK + dane innego użytkownika
```

## Remediacja developerska

Dobre praktyki backendowe:

- nie ufać `userId`, `accountId`, `fileId`, `invoiceId`, `role` ani `isAdmin` z requestu,
- brać obecnego użytkownika z sesji albo tokena,
- sprawdzać ownership przed zwróceniem danych,
- sprawdzać permissions przy każdej wrażliwej akcji,
- stosować principle of least privilege,
- dodawać automatyczne testy kontroli dostępu,
- logować podejrzane próby dostępu do cudzych zasobów.

Ryzykowny wzorzec:

```ts
getInvoice(req.query.invoiceId)
```

Bezpieczniejszy wzorzec:

```ts
getInvoiceForUser(req.session.userId, req.query.invoiceId)
```

Backend musi sprawdzić zarówno, jaki obiekt został zażądany, jak i czy obecny użytkownik może go zobaczyć lub zmienić.

## Review Checklist

- Czy request zawiera referencję do obiektu?
- Czy użytkownik może ją zmienić?
- Czy obecny użytkownik pochodzi z sesji/tokena, a nie z request body?
- Czy backend sprawdza ownership?
- Czy endpoint zachowuje się poprawnie dla drugiego użytkownika?
- Czy frontend tylko ukrywa akcję, którą backend nadal pozwala wykonać?
- Czy poprawnym response powinno być `403`, `404` albo redirect do logowania?

## Główna lekcja

Frontend nie jest granicą bezpieczeństwa.

React może ukryć przycisk albo link, ale użytkownik nadal może wysłać zmodyfikowany request przez Burpa, Postmana, curl albo skrypt.

Backend zawsze musi odpowiedzieć:

> Czy ten zalogowany użytkownik ma prawo dostać się do dokładnie tego obiektu i wykonać dokładnie tę akcję?
