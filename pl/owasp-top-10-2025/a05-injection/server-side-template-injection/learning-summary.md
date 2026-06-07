# SSTI Learning Summary

## Co ukonczylem

Ukonczylem teorie i praktyke SSTI dla czterech kombinacji engine/runtime:

- Smarty / PHP w TryHackMe,
- Pug / Node.js w TryHackMe,
- Jinja2 / Python w TryHackMe,
- ERB / Ruby w PortSwigger Web Security Academy.

Przejrzalem tez SSTImap jako narzedzie automatyzacji dla autoryzowanych testow.

## Moje poczatkowe rozumienie

Rozumialem ogolny przeplyw:

```text
user-controlled input -> server-side template engine -> evaluated output
```

Poczatkowo jednak zbyt szybko opisywalem SSTI jako arbitrary code execution i zbyt mocno polegalem na "sanitise the input" jako poprawce.

Poprawiony model:

- prosta ewaluacja potwierdza SSTI,
- glebszy dostep trzeba udowodnic osobno,
- glowna poprawka to trzymanie inputu uzytkownika poza template source.

## Trudnosc: process APIs

### Pug / Node.js

Poczatkowo zle rozumialem `spawnSync()` i probowalem:

- przekazac komende plus argumenty jako jeden executable string,
- uzyc trzeciego parametru jako kolejnej komendy.

Nauczylem sie:

```javascript
spawnSync(command, args, options)
```

Trzeci parametr to konfiguracja, nie drugi proces. Argumenty powinny byc w tablicy.

### Jinja2 / Python

Ten sam koncept zobaczylem przy `subprocess.check_output()`:

```python
check_output('ls')
```

moze dzialac dla komendy bez argumentow, a:

```python
check_output('ls -lah')
```

zwykle zawiedzie przy `shell=False`, bo string jest traktowany jako nazwa executable. Poprawna struktura:

```python
check_output(['ls', '-lah'])
```

To pomoglo mi zrozumiec roznice miedzy shell parsing a direct process APIs.

## Trudnosc: payloady zalezne od srodowiska

Na poczatku dlugie object-traversal chains wygladaly jak reusable SSTI syntax.

Nauczylem sie, ze sciezki uzywajace:

- Jinja `__mro__` / `__subclasses__()` i hard-coded indexes,
- Node.js `process.mainModule`,

zaleza od wersji, importow, runtime mode, dostepnych obiektow i konfiguracji. To techniki labowe, nie uniwersalne payloady.

## PortSwigger ERB lab: moj proces rozumowania

### Poczatkowa pomylka

Najpierw wskazalem `productId` jako kontrolowany parametr, bo patrzylem na product requests.

Po sprawdzeniu flow out-of-stock znalazlem faktyczny parametr:

```text
message
```

### Reflection test

Podmienilem message na:

```text
SSTI_TEST_123
```

Marker pojawil sie w `<div>` w odpowiedzi.

To potwierdzilo:

- kontroluje `message`,
- wiem, gdzie jest renderowany.

Nie potwierdzilo jeszcze SSTI.

### Evaluation test

Uzylem nieszkodliwego wyrazenia arytmetycznego ERB. Odpowiedz zawierala:

```text
49
```

To potwierdzilo, ze serwer ocenil wyrazenie przez ERB, zamiast wyswietlic je literalnie.

### Niejasnosc sciezki pliku

Nie wiedzialem, skad wziela sie sciezka pliku docelowego. Nauczylem sie, ze sciezka byla podana bezposrednio w opisie labu. Ten lab testowal ERB evaluation i uzycie Ruby API, nie filesystem enumeration.

### Luka w Ruby

Nie znalem Ruby file API. Lab uzyl:

```ruby
File.delete(path)
```

To bylo bezposrednie API Ruby i nie wymagalo shella ani `rm`.

### Root cause

Poczatkowo pytalem, co znaczy root cause.

Finalne wyjasnienie:

> Aplikacja wstawila kontrolowana przez uzytkownika wartosc `message` do ERB template source i wyrenderowala ja. Input stal sie template code zamiast pozostac danymi.

### Korekta remediacji

Moja pierwsza odpowiedz skupiala sie na sanitizacji i czyszczeniu inputu.

Poprawione podejscie:

- nie buduj template source z inputu uzytkownika,
- trzymaj templates jako statyczne i kontrolowane przez developerow,
- przekazuj `message` tylko jako data variable,
- gdy mozliwe, przyjmuj status ID i wybieraj staly komunikat po stronie serwera.

### Luka w testach regresji

Poczatkowo nie umialem zaproponowac testu regresji.

Nauczylem sie prostego testu:

```text
send template expression
  -> it must remain text or be rejected
  -> it must not become the calculated result
```

## Co teraz umiem wyjasnic

- czym zajmuje sie server-side template engine,
- czym SSTI rozni sie od reflection, XSS i OS Command Injection,
- dlaczego obliczenie potwierdza ewaluacje, ale nie automatycznie RCE,
- jak error messages i zachowanie wyrazen pomagaja fingerprintowac engines,
- dlaczego struktura argumentow process API ma znaczenie,
- dlaczego sandboxing jest defence in depth,
- dlaczego dane uzytkownika nie moga stac sie template source,
- jak napisac podstawowy test regresji SSTI.

## Aktualna ocena

**PASS**

To praktyczna i poprawna podstawa dla Frontend Engineera przechodzacego w AppSec. Nie potrzebuje kolejnego labu SSTI przed kontynuacja planu A05. Dodatkowe laby moga sluzyc pozniej do powtorki, sandbox escape concepts albo glebszego wejscia w konkretny engine.
