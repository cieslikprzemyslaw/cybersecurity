# PortSwigger Lab 02 - podstawowy SSRF na inny backend

## Lab

PortSwigger Web Security Academy:

```text
Basic SSRF against another back-end system
```

## Co testowałem

Testowałem ten sam typ funkcji sprawdzania stanu magazynowego, ale tym razem stock API wskazywało na wewnętrzny prywatny adres IP:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://192.168.0.1:8080/product/stock/check?productId=1&storeId=1
```

Normalna odpowiedź zwracała liczbę sztuk w magazynie.

To pokazało, że backend aplikacji mógł komunikować się z wewnętrznym backendem w sieci `192.168.0.X`.

## Co znalazłem

Lab wymagał znalezienia panelu admina gdzieś w wewnętrznym zakresie:

```text
192.168.0.X:8080
```

Zamiast ręcznie testować każdy IP, użyłem Burp Intruder do iterowania po ostatnim oktecie.

Wzorzec koncepcyjny:

```text
http://192.168.0.§1§:8080/admin
```

Zakres payloadu:

```text
1 to 255
```

Porównywałem odpowiedzi przez:

- status code,
- długość odpowiedzi,
- tytuł HTML,
- body odpowiedzi,
- obecność treści związanej z panelem admina.

Jeden IP zwrócił panel admina z linkami usuwania użytkowników.

Po zidentyfikowaniu wewnętrznego hosta admina użyłem podatnego parametru `stockApi`, żeby backend wykonał request do wewnętrznego endpointu usuwania użytkownika `carlos`.

## Ważny moment nauki

Nie od razu pomyślałem o Burp Intruder.

Przydatny skrót myślowy:

> SSRF + wewnętrzny zakres IP = użyj Intrudera w legalnym labie.

To różni się od pierwszego labu, bo celem nie był `localhost`. Podatny backend mógł dotrzeć do innej usługi w sieci prywatnej.

## Dlaczego to ma znaczenie

Ten lab pokazał, że SSRF nie ogranicza się do samego podatnego serwera.

Podatny backend może stać się proxy do sieci wewnętrznej.

Jeżeli usługi wewnętrzne ufają lokalizacji sieciowej serwera aplikacji, SSRF może dać dostęp do wrażliwej funkcjonalności niedostępnej z internetu.

## Przyczyna źródłowa

Przyczyną było:

- miejsce docelowe requestu backendowego kontrolowane przez użytkownika,
- dostęp backendu do prywatnego zakresu IP,
- brak ścisłej allowlisty dla celów stock API,
- wewnętrzna usługa admina osiągalna z backendu aplikacji,
- usługa wewnętrzna polegająca na izolacji sieciowej zamiast silnego authentication.

## Wpływ

Możliwy wpływ:

- odkrywanie wewnętrznych hostów,
- dostęp do wewnętrznych paneli admina,
- nieautoryzowane akcje na backendach,
- rekonesans sieci wewnętrznej,
- ujawnienie usług niewystawionych do internetu,
- obejście założeń opartych na lokalizacji sieciowej.

## Rekonesans sieci wewnętrznej

SSRF może służyć do wnioskowania o systemach wewnętrznych przez testowanie:

```text
różnych adresów IP
różnych portów
różnych ścieżek
różnych protokołów
```

Sygnały:

```text
różne status codes
różne długości odpowiedzi
timeouty
błędy
strony HTML
redirecty
konkretne słowa kluczowe
```

Dlatego SSRF może być poważniejszy niż prosty problem z zewnętrznym requestem.

## Naruszona zasada AppSec

Aplikacja naruszyła tę granicę zaufania:

> Publiczny request kontrolowany przez użytkownika nie powinien pozwalać backendowi kontaktować się z dowolnymi usługami w sieci wewnętrznej.

## Remediacja

Bezpieczniejsza implementacja powinna:

- nie pozwalać użytkownikom kontrolować URL-i usług backendowych,
- używać mapowań po stronie serwera dla usług stock,
- pozwalać tylko na znane hosty stock API,
- blokować prywatne zakresy IP, chyba że są jawnie wymagane,
- ograniczać porty,
- walidować DNS resolution i finalny IP,
- ponownie sprawdzać cel po redirectach,
- izolować wewnętrzne systemy admina od workloadów aplikacji,
- wymagać authentication i authorization na usługach wewnętrznych.

## Pomysł na test regresji

Dodaj testy upewniające się, że `stockApi` odrzuca cele w sieci prywatnej:

```text
http://192.168.0.1:8080/admin
http://192.168.0.127:8080/admin
http://10.0.0.1:8080/admin
http://172.16.0.1:8080/admin
```

Oczekiwany wynik:

```text
Backend nie może służyć jako proxy do skanowania albo odpytywania hostów wewnętrznych.
```

## Lekcja dla developera

SSRF może zmienić backend w most do sieci wewnętrznej.

Bezpieczna poprawka nie polega tylko na blokowaniu `localhost`. Aplikacja musi kontrolować i walidować wszystkie miejsca docelowe requestów backendowych, w tym prywatne zakresy IP i finalne cele po redirectach.
