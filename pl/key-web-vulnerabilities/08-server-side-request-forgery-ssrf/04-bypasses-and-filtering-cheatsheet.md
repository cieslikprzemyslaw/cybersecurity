# SSRF - bypassy i filtrowanie

## Cel

Ta notatka pomaga zrozumieć, dlaczego proste filtry SSRF często zawodzą.

To nie jest dump payloadów. To zbiór wzorców z legalnych labów i koncepcji przydatnych przy remediacji z perspektywy developera.

## Dlaczego blacklisty są słabe

Blacklista próbuje blokować znane złe wartości, np.:

```text
localhost
127.0.0.1
/admin
```

To kruche podejście, bo atakujący może użyć równoważnych reprezentacji, które prowadzą do tego samego celu, ale nie pasują do blokowanego stringa.

Bezpieczniejszy design to ścisłe allowlisty, walidacja finalnego rozwiązanego celu i ograniczenia sieciowe.

## Wzorzec 1: blokowanie `localhost`

Zablokowany input:

```text
http://localhost/admin
```

Możliwy problem:

```text
Filtr blokuje dosłowne słowo "localhost".
```

To niekoniecznie blokuje inne reprezentacje loopback.

## Wzorzec 2: blokowanie `127.0.0.1`

Zablokowany input:

```text
http://127.0.0.1/admin
```

Możliwy bypass w labach:

```text
http://127.1/
```

Dlaczego to ma znaczenie:

```text
127.1 nadal może wskazywać na loopback, ale słaby filtr może blokować tylko dokładny string 127.0.0.1.
```

Inne równoważne reprezentacje IP również mogą istnieć, zależnie od parsera i środowiska.

## Wzorzec 3: blokowanie wrażliwych ścieżek

Zablokowany input:

```text
/admin
```

Możliwy problem:

```text
Filtr szuka dosłownego stringa "admin".
```

Jeżeli backend dekoduje URL później, zakodowane znaki nadal mogą trafić do docelowej ścieżki.

Przykład koncepcyjny:

```text
/admin
/%61dmin
/%2561dmin
```

## Wzorzec 4: double encoding

Double encoding może ominąć filtry, gdy walidacja i przetwarzanie requestu dekodują input inaczej.

Przykład:

```text
%2561
```

Pierwsze dekodowanie:

```text
%61
```

Drugie dekodowanie:

```text
a
```

Więc:

```text
/%2561dmin
```

może finalnie stać się:

```text
/admin
```

Słaba blacklista może nie wykryć finalnej ścieżki, jeśli waliduje input przed pełnym dekodowaniem i normalizacją.

## Wzorzec 5: confusion między parserami URL

Różne parsery mogą interpretować ten sam URL inaczej.

Przykłady koncepcyjne:

```text
https://expected-host@evil-host
https://evil-host#expected-host
https://expected-host.evil-host
```

Ryzyko:

```text
Kod walidujący może uznać, że URL wskazuje dozwolony host, podczas gdy klient HTTP odpytuje inny finalny host.
```

## Wzorzec 6: bypass przez redirect

Jeżeli backend podąża za redirectami, dozwolony URL może przekierować do zablokowanego wewnętrznego URL-a.

Flow:

```text
Backend waliduje dozwolony URL
Dozwolony URL zwraca redirect
Backend podąża za redirectem do celu wewnętrznego
```

Ryzyko:

```text
Aplikacja waliduje pierwszy URL, ale nie finalny cel po redirectach.
```

## Wzorzec 7: open redirect + SSRF

Jeżeli dozwolony endpoint aplikacji zawiera open redirect, można go połączyć z SSRF.

Koncepcyjny flow:

```text
stockApi=https://allowed-host/redirect?path=http://internal-target/admin
```

URL może przejść allowlistę, bo zaczyna się od dozwolonego hosta albo należy do niego, ale backend finalnie podąża za redirectem do celu wewnętrznego.

## Wzorzec 8: problemy DNS

Potencjalne ryzyka DNS w SSRF:

- domeny kontrolowane przez atakującego rozwiązujące się do wewnętrznych IP,
- DNS rebinding,
- walidacja przed rozwiązaniem DNS,
- różne wyniki DNS między walidacją a czasem requestu.

Notatka dla developera:

```text
Nie waliduj tylko stringa hostname. Waliduj rozwiązany IP i finalny cel w momencie requestu.
```

## Co porównywać podczas testowania filtrów

Podczas mapowania filtrów SSRF porównuj:

```text
localhost
127.0.0.1
127.1
private IP ranges
allowed external host
blocked path
encoded path
double-encoded path
different protocols
different ports
redirect behaviour
```

Zapisuj:

```text
Status code
Response body
Response length
Error message
Redirect location
```

## Lekcja dla developera

Filtr typu:

```text
block if input contains "localhost" or "admin"
```

nie jest solidną ochroną przed SSRF.

Bezpieczniejszy wzorzec:

```text
bezpiecznie parsuj URL
rozwiązuj hostname
waliduj scheme
waliduj port
waliduj finalny IP
blokuj private/loopback/link-local/metadata ranges
podążaj za redirectami tylko, jeśli każdy hop jest walidowany
pozwalaj tylko na konkretne oczekiwane cele
wymuszaj network-level egress controls
```

## Główna lekcja

Ochrona SSRF oparta na blacklistach zawodzi, bo zwykle waliduje stringi, a nie finalny cel sieciowy.

Najważniejsze pytanie bezpieczeństwa brzmi:

> Dokąd backend faktycznie wyśle request po parsowaniu, dekodowaniu, DNS resolution, redirectach i normalizacji?
