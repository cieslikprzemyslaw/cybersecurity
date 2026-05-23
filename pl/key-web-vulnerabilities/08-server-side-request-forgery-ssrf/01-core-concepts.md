# SSRF - podstawowe pojęcia

## TL;DR

Server-Side Request Forgery (SSRF) występuje wtedy, gdy atakujący może wpłynąć na to, dokąd aplikacja po stronie serwera wysyła request HTTP.

Kluczowy punkt:

> Request wykonuje backend, a nie bezpośrednio przeglądarka użytkownika.

To ma znaczenie, bo backend może mieć dostęp do usług wewnętrznych, interfejsów dostępnych tylko z `localhost`, prywatnych zakresów IP, endpointów cloud metadata albo innych systemów niedostępnych publicznie z internetu.

## Podstawowy model myślenia

Normalny request z przeglądarki:

```text
przeglądarka użytkownika -> publiczna aplikacja webowa
```

Przepływ SSRF:

```text
użytkownik kontroluje input -> backend aplikacji -> backend wysyła request do wskazanego celu
```

W SSRF atakujący nie łączy się bezpośrednio z wewnętrznym celem. Nadużywa zaufanej pozycji sieciowej serwera aplikacji.

## Dlaczego SSRF jest server-side

SSRF jest podatnością server-side, ponieważ aplikacja bierze input kontrolowany przez użytkownika i używa go do wykonania requestu backendowego.

Przykład:

```http
POST /product/stock

stockApi=http://stock.example.internal/product/stock/check?productId=1&storeId=1
```

Jeśli backend pobiera wartość `stockApi`, to zmiana tej wartości może zmienić miejsce, do którego serwer wyśle request.

Niebezpieczne nie jest samo to, że parametr zawiera URL. Niebezpieczne jest to, że backend ufa temu URL-owi i go pobiera.

## Dlaczego `localhost` oznacza serwer

W SSRF te adresy nie oznaczają laptopa użytkownika:

```text
http://localhost
http://127.0.0.1
```

Oznaczają `localhost` serwera aplikacji.

Jeżeli request backendowy zostanie zmieniony na:

```text
http://localhost/admin
```

serwer aplikacji może odpytać własny wewnętrzny panel administracyjny.

Może to ominąć kontrolę dostępu, która błędnie ufa requestom pochodzącym z lokalnej maszyny.

## URL zewnętrzny vs URL wewnętrzny

URL zewnętrzny jest publicznie osiągalny:

```text
https://example.com/image.jpg
https://api.public-service.com/data
```

URL wewnętrzny może być osiągalny tylko z infrastruktury aplikacji:

```text
http://localhost/admin
http://127.0.0.1/admin
http://10.0.0.5/internal-api
http://192.168.0.10:8080/admin
```

Użytkownik może nie mieć bezpośredniego dostępu do wewnętrznego URL-a, ale backend może go mieć.

## Prywatne i wewnętrzne zakresy IP

Warto pamiętać o tych zakresach:

```text
127.0.0.0/8       loopback / localhost
10.0.0.0/8        sieć prywatna
172.16.0.0/12     sieć prywatna
192.168.0.0/16    sieć prywatna
169.254.0.0/16    adresy link-local
```

Ważny endpoint cloud metadata:

```text
169.254.169.254
```

Ten adres jest często kojarzony z usługami metadata instancji cloud. Jeżeli podatny backend może go osiągnąć, SSRF może ujawnić metadata instancji, tokeny, informacje o rolach IAM albo inną wrażliwą konfigurację, zależnie od środowiska i zabezpieczeń.

## Regular SSRF vs blind SSRF

### Regular / non-blind SSRF

Regular SSRF występuje wtedy, gdy odpowiedź backendu jest zwracana w odpowiedzi aplikacji.

Przykład:

```text
stockApi=http://localhost/admin
```

Aplikacja zwraca HTML wewnętrznej strony administracyjnej.

To jest łatwiejsze do wykrycia, bo tester widzi wynik bezpośrednio.

### Blind SSRF

Blind SSRF występuje wtedy, gdy backend wykonuje request, ale odpowiedź nie jest zwracana użytkownikowi.

Detekcja może wymagać pośrednich sygnałów, takich jak:

- callback do kontrolowanego serwera,
- interakcja w Burp Collaborator,
- różnice czasowe,
- inne komunikaty błędów,
- inne status codes.

Blind SSRF jest trudniejszy do udowodnienia, ale nadal może być poważny.

## Dlaczego SSRF jest groźny

SSRF może pozwolić atakującemu:

- dostać się do paneli administracyjnych dostępnych tylko z `localhost`,
- uzyskać dostęp do wewnętrznych API,
- dotrzeć do backendów na prywatnych adresach IP,
- wykonywać rekonesans wewnętrznej sieci,
- odpytać endpointy cloud metadata,
- wywołać nieautoryzowane akcje zmieniające stan,
- wyciec dane z zaufanych usług wewnętrznych.

Rzeczywisty wpływ zależy od tego, co podatny backend może osiągnąć.

## SSRF w porównaniu z poprzednimi tematami

Path traversal:

```text
Atakujący kontroluje ścieżkę pliku używaną przez serwer.
```

File upload:

```text
Atakujący kontroluje plik zapisywany lub przetwarzany przez serwer.
```

SSRF:

```text
Atakujący kontroluje cel, do którego serwer wysyła request.
```

W SSRF payloadem jest często URL, host, ścieżka albo wewnętrzny cel, a nie ścieżka pliku, uploadowany plik albo fragment SQL.

## Główna lekcja

SSRF nadużywa zaufania po stronie serwera.

Atakujący nie potrzebuje bezpośredniego dostępu do systemu wewnętrznego. Wystarczy mu sposób, żeby zmusić zaufany backend do wysłania requestu w jego stronę.
