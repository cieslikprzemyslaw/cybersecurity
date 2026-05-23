# PortSwigger Lab 01 - podstawowy SSRF na lokalny serwer

## Lab

PortSwigger Web Security Academy:

```text
Basic SSRF against the local server
```

## Co testowałem

Testowałem funkcję sprawdzania stanu magazynowego, w której frontend wysyłał parametr `stockApi` do backendu.

Normalny request wyglądał jak standardowe sprawdzenie stanu produktu:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

Normalna odpowiedź zwracała liczbę sztuk w magazynie.

To sugerowało, że backend pobiera URL przekazany w `stockApi` i zwraca wynik użytkownikowi.

## Co znalazłem

Zmiana wartości `stockApi` na URL localhost sprawiła, że backend odpytał wewnętrzną stronę admina:

```http
stockApi=http://localhost/admin
```

Odpowiedź zwróciła HTML panelu administracyjnego.

To potwierdziło regular / non-blind SSRF, ponieważ odpowiedź backendu była widoczna w odpowiedzi frontendu.

Panel admina zawierał link usuwania użytkownika `carlos`.

Akcja zmieniająca stan działała tylko wtedy, gdy była wywołana przez podatny parametr `stockApi`, ponieważ request wykonywał backend z kontekstu server-side.

## Ważny błąd / moment nauki

Najpierw próbowałem wywołać endpoint usuwania bezpośrednio:

```http
/admin/delete?username=carlos
```

To zwróciło błąd autoryzacji, bo request pochodził z mojego kontekstu przeglądarki/użytkownika.

Najważniejsza lekcja:

> Request usuwający musiał przejść przez podatny parametr backendowy, a nie bezpośrednio z przeglądarki.

Poprawny model:

```text
Źle:
browser -> /admin/delete

Dobrze:
browser -> /product/stock -> backend -> http://localhost/admin/delete
```

## Dlaczego to ma znaczenie

Panel admina nie był bezpiecznie chroniony. Ufał requestom pochodzącym z `localhost`.

SSRF pozwolił zewnętrznemu użytkownikowi zmusić backend do wysłania requestu do własnego lokalnego interfejsu.

To ominęło zamierzone ograniczenie dostępu.

## Przyczyna źródłowa

Przyczyną była kombinacja:

- miejsca docelowego requestu backendowego kontrolowanego przez użytkownika,
- pobierania dowolnych URL-i z `stockApi` przez backend,
- funkcji admina ufającej requestom z `localhost`,
- braku właściwej walidacji celu,
- braku właściwego authentication i authorization dla wewnętrznej akcji admina.

## Wpływ

Możliwy wpływ:

- dostęp do panelu admina dostępnego tylko z `localhost`,
- nieautoryzowane akcje administracyjne,
- usuwanie użytkowników,
- obejście kontroli dostępu opartych na lokalizacji sieciowej,
- ujawnienie wewnętrznej funkcjonalności.

## Naruszona zasada AppSec

Aplikacja naruszyła podstawową granicę zaufania:

> Input kontrolowany przez użytkownika nie powinien definiować dowolnych requestów server-side.

Opierała się też na niebezpiecznym zaufaniu:

> Request z `localhost` nie jest automatycznie bezpieczny.

## Remediacja

Bezpieczniejsza implementacja powinna:

- nie przyjmować dowolnych URL-i w `stockApi`,
- używać mapowań po stronie serwera dla znanych usług stock,
- pozwalać tylko na oczekiwane cele stock API,
- blokować `localhost`, loopback, private, link-local i metadata IP ranges,
- walidować finalny rozwiązany cel,
- wyłączyć albo walidować redirecty,
- wymagać prawdziwego authentication i authorization dla akcji admina,
- nie polegać na `localhost` ani sieci wewnętrznej jako kontroli dostępu.

## Pomysł na test regresji

Dodaj testy upewniające się, że endpoint stock check odrzuca:

```text
http://localhost/admin
http://127.0.0.1/admin
http://127.1/admin
http://[::1]/admin
```

Oczekiwany wynik:

```text
Backend nie może odpytywać wewnętrznych endpointów admina przez stockApi.
```

## Lekcja dla developera

Jeżeli funkcja backendowa pobiera URL przekazany przez użytkownika, może stać się mostem do systemów wewnętrznych.

Wewnętrzne panele admina nie mogą ufać samemu `localhost`, ponieważ SSRF może sprawić, że request kontrolowany przez atakującego będzie wyglądał jak request z `localhost`.
