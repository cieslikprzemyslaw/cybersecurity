# Burp Suite, Proxy i podstawy Repeatera

## 1. Overview

Ta notatka opisuje podstawowy workflow pracy z Burp Suite podczas testowania bezpieczeństwa aplikacji webowych.

Burp Suite pomaga podglądać i modyfikować komunikację między przeglądarką a serwerem. To ważne, ponieważ wiele podatności nie jest widocznych bezpośrednio w UI. Stają się widoczne dopiero wtedy, gdy patrzę na prawdziwe requesty i response'y HTTP.

Podstawowy przepływ wygląda tak:

```text
Browser -> Burp Proxy -> Server
Server -> Burp Proxy -> Browser
```

Burp pozwala spowolnić, podejrzeć, zmienić i ponownie wysłać requesty ręcznie.

## 2. Czym jest Burp Suite?

Burp Suite to narzędzie do testowania bezpieczeństwa aplikacji webowych.

Można go używać do:

- przechwytywania requestów HTTP,
- analizowania response'ów HTTP,
- modyfikowania requestów zanim trafią do serwera,
- ponownego wysyłania requestów,
- testowania parametrów,
- testowania cookies i tokenów,
- porównywania odpowiedzi serwera,
- zrozumienia, jak aplikacja komunikuje się z backendem.

W nauce AppSec Burp jest przydatny, bo pokazuje, co naprawdę dzieje się za UI.

## 3. Proxy Concept

Proxy to pośrednik między klientem a serwerem.

Gdy Burp jest skonfigurowany jako proxy:

1. Przeglądarka wysyła request.
2. Request trafia najpierw do Burpa.
3. Burp może go pokazać, zatrzymać albo zmienić.
4. Request jest przekazywany do serwera.
5. Response serwera wraca przez Burpa.
6. Przeglądarka otrzymuje response.

Przepływ:

```text
Browser -> Burp -> Server
Browser <- Burp <- Server
```

Dzięki temu mogę analizować zachowanie aplikacji na poziomie HTTP.

## 4. Intercept

Intercept pozwala zatrzymać request zanim dotrze do serwera.

Gdy `Intercept is on`, Burp zatrzymuje request.

W tym momencie mogę:

- przeczytać request,
- zmienić request,
- wysłać go dalej,
- odrzucić go.

| Action | Znaczenie |
|---|---|
| `Forward` | Wysyła request do serwera |
| `Drop` | Anuluje request |
| `Intercept is on` | Burp będzie zatrzymywał pasujące requesty |
| `Intercept is off` | Requesty przechodzą normalnie |

Jeśli nie kliknę `Forward`, przeglądarka może cały czas ładować stronę, ponieważ request czeka w Burpie.

Intercept jest przydatny, gdy chcę złapać konkretną akcję, np.:

- logowanie,
- aktualizację profilu,
- reset hasła,
- checkout,
- upload pliku,
- akcję usuwania,
- akcję admina.

## 5. Repeater

Repeater służy do ręcznego ponownego wysyłania i modyfikowania requestów.

To jedno z najważniejszych narzędzi w Burpie podczas nauki AppSec, ponieważ mogę wysyłać ten sam request wiele razy i porównywać odpowiedzi serwera.

Przykład:

```http
GET /product?id=1 HTTP/1.1
Host: example.com
```

Zmienione w Repeaterze:

```http
GET /product?id=2 HTTP/1.1
Host: example.com
```

To pomaga sprawdzić, czy serwer poprawnie kontroluje dostęp do zasobów.

Repeater jest przydatny do testowania:

- IDOR,
- Broken Access Control,
- zachowania authentication,
- sprawdzania authorization,
- walidacji inputu,
- SQL Injection,
- payloadów XSS,
- parameter tampering,
- zachowania cookies i sesji.

## 6. Co można zmienić w requestcie?

W Burp Repeater można zmienić wiele części requestu.

Przykłady:

- metodę HTTP,
- ścieżkę URL,
- query parameters,
- request body,
- headers,
- cookies,
- tokeny,
- content type,
- ID zasobów,
- ID użytkowników,
- ID kont,
- wartości związane z rolami,
- wartości boolean, np. `isAdmin`.

Przykładowy request:

```http
GET /profile?id=1 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Możliwe zmiany:

```http
GET /profile?id=2 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Albo:

```http
POST /profile?id=1 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Albo:

```http
DELETE /profile/1 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Celem nie jest losowa zmiana requestów. Celem jest zrozumienie, czy serwer poprawnie waliduje request i sprawdza uprawnienia.

## 7. Testowanie zachowania serwera

Po zmianie requestu serwer może odpowiedzieć na różne sposoby.

| Response | Możliwe znaczenie |
|---|---|
| `200 OK` | Request zakończony sukcesem |
| `302 Found` | Redirect, często do logowania albo innej strony |
| `400 Bad Request` | Niepoprawny format requestu albo input |
| `401 Unauthorized` | Użytkownik nie jest zalogowany albo sesja jest niepoprawna |
| `403 Forbidden` | Użytkownik jest zalogowany, ale nie ma uprawnień |
| `404 Not Found` | Zasób nie istnieje albo jest ukryty |
| `405 Method Not Allowed` | Metoda HTTP nie jest akceptowana |
| `500 Internal Server Error` | Błąd po stronie serwera |

Bezpieczny serwer nie musi zaakceptować zmodyfikowanego requestu.

Na przykład, jeśli zmienię:

```http
GET /api/users/123
```

na:

```http
DELETE /api/users/123
```

bezpieczny serwer powinien odrzucić taki request, chyba że użytkownik ma prawo usunąć ten zasób.

## 8. Testowanie cookies i sesji

Cookies są często ważne podczas testowania bezpieczeństwa aplikacji webowych, ponieważ mogą zawierać identyfikatory sesji.

Przykład:

```http
Cookie: session=abc123
```

W Burpie mogę sprawdzić, co się stanie, jeśli zmienię cookie:

```http
Cookie: session=test
```

Możliwe zachowanie serwera:

- redirect do logowania,
- `401 Unauthorized`,
- `403 Forbidden`,
- utworzenie nowej anonimowej sesji,
- zignorowanie niepoprawnego cookie,
- zwrócenie błędu,
- zaakceptowanie cookie tylko wtedy, gdy jest poprawne i aktywne.

Najważniejsze pytanie:

> Czy serwer poprawnie weryfikuje sesję, czy ślepo ufa cookie?

## 9. Dlaczego Burp jest ważny w AppSec?

Burp pomaga myśleć poza widoczną stroną.

Zwykły użytkownik klika przyciski, formularze i linki. AppSec tester patrzy na request stojący za tymi akcjami.

Na przykład przycisk może wywołać taki request:

```http
POST /api/account/update-email
```

Ważne pytania AppSec:

- Czy mogę zmienić email na email innego użytkownika?
- Czy mogę zmienić user ID?
- Czy mogę zmienić rolę?
- Czy mogę usunąć wymagane pola?
- Czy mogę zmienić metodę?
- Czy mogę użyć requestu bez ważnej sesji?
- Czy mogę wykonać akcję jako inny użytkownik?

Burp pozwala sprawdzić te pytania w praktyce.

## 10. Request Review Checklist

Analizując request w Burpie, powinienem sprawdzić:

- Jaki endpoint jest wywoływany?
- Jaka metoda HTTP jest używana?
- Czy są query parameters?
- Czy jest request body?
- Czy są ID, np. `userId`, `accountId`, `orderId` albo `profileId`?
- Czy są wartości związane z rolą, np. `role`, `admin`, `isAdmin`?
- Czy są cookies albo tokeny?
- Jaki content type jest używany?
- Jaki status response jest zwracany?
- Czy response body zawiera wrażliwe dane?
- Czy serwer sprawdza, czy ten użytkownik powinien mieć prawo wykonać tę akcję?

Przydatny mindset:

> Jeśli wartość identyfikuje użytkownika, konto, obiekt albo uprawnienie, warto ją ostrożnie przetestować.

## 11. Key Takeaways

- Burp działa jako proxy między przeglądarką a serwerem.
- Intercept zatrzymuje request zanim dotrze do serwera.
- Forward wysyła request dalej.
- Drop anuluje request.
- Repeater pozwala ręcznie testować ten sam request wiele razy.
- Parametry, cookies, headers, body i metody requestu można modyfikować.
- Serwer powinien odrzucać requesty, które są niepoprawne lub nieautoryzowane.
- Status codes i response body mają znaczenie.
- Burp pomaga zobaczyć, jak aplikacja działa za UI.
- Ręczne testowanie requestów to podstawowa umiejętność AppSec.
