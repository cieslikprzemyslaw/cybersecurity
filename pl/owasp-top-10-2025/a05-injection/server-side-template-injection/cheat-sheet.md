# SSTI Cheat Sheet

> Tylko dla TryHackMe, PortSwigger, lokalnych labow i jawnie autoryzowanych testow.

## Core model

```text
user input -> template source -> template engine -> evaluated behaviour
```

## Pierwsze pytania

1. Jaka funkcja generuje output?
2. Jaki konkretnie input kontroluje?
3. Gdzie sie pojawia?
4. Czy jest odbity literalnie?
5. Czy jest HTML-encoded?
6. Jaki engine albo runtime podejrzewam?
7. Jakie dowody wspieraja te hipoteze?
8. Jaki pojedynczy, nieszkodliwy test potwierdzilby ewaluacje?

## Drabina dowodow

```text
marker reflected
  -> tylko kontrola inputu

expression returns calculated result
  -> template evaluation confirmed

engine-specific error or behaviour
  -> fingerprinting evidence

context/secret/file access
  -> deeper impact confirmed

process execution or destructive side effect
  -> critical impact confirmed
```

## Nie myl

- Reflection nie jest dowodem SSTI.
- Renderowanie HTML nie oznacza automatycznie SSTI.
- SSTI jest server-side; XSS jest browser-side.
- SSTI prowadzace do procesu nie zmienia root cause w command injection.
- Wynik matematyczny potwierdza ewaluacje, nie full RCE.
- Wynik skanera nie zastepuje dowodow.

## Przypomnienia z ukonczonych labow

### Smarty

- Ekosystem PHP.
- Modifiers/functions zaleza od security policy i konfiguracji.

### Pug

- Node.js / JavaScript.
- `#{...}` ocenia i escapuje output.
- `!{...}` ocenia i generuje unescaped output.
- `spawnSync(command, args, options)`.

### Jinja2

- Jezyk szablonow podobny do Pythona.
- Object traversal chains sa zalezne od srodowiska.
- `check_output(['ls', '-lah'])` rozdziela executable i argumenty.

### ERB

- Wyrazenia Ruby.
- Obliczony wynik moze potwierdzic ewaluacje.
- API Ruby takie jak `File` moze pokazac impact w autoryzowanym labie.

## Przypomnienia process API

### Node.js

```javascript
spawnSync('ls', ['-lah'])
```

- Trzeci parametr to obiekt opcji.
- `stdout` jest Bufferem.

### Python

```python
subprocess.check_output(['ls', '-lah'])
```

- `shell=False` jest domyslne.
- Output jest bytes, jesli nie wlaczysz text/encoding.

## Root cause statement

> Input kontrolowany przez uzytkownika stal sie czescia wykonywalnego template source, zamiast pozostac zwykla wartoscia przekazana do statycznego szablonu.

## Main remediation statement

> Trzymaj template source jako statyczne i kontrolowane przez developerow. Input uzytkownika przekazuj tylko jako template data albo variables; nie kompiluj go ani nie ewaluuj ponownie.

## Dyscyplina testowania

Przed kazdym testem:

- hipoteza,
- powod,
- oczekiwany wynik,
- znaczenie innego wyniku.

Po kazdym tescie:

- co sie zmienilo,
- co zostalo bez zmian,
- literalne czy ocenione,
- bledy,
- dowody,
- kolejna hipoteza.
