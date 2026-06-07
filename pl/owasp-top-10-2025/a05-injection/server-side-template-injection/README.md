# Server-Side Template Injection

Server-Side Template Injection (SSTI) wystepuje wtedy, gdy niezaufany input staje sie czescia zrodla szablonu po stronie serwera i zostaje oceniony przez template engine.

## Model mentalny

```text
input kontrolowany przez uzytkownika
  -> server-side template source
  -> template engine
  -> ewaluacja wyrazenia albo zmienione renderowanie
  -> mozliwy dostep do obiektow aplikacji, plikow, procesow albo sekretow
```

Kluczowy problem nie polega tylko na tym, ze input pojawia sie w HTML. Problem polega na tym, ze serwer traktuje ten input jak skladnie szablonu, a nie jak zwykle dane.

## Ukończona nauka

- TryHackMe: teoria i praktyka z:
  - Smarty / PHP,
  - Pug / Node.js,
  - Jinja2 / Python.
- Swiadomosc SSTImap i uzycie w autoryzowanym labie.
- PortSwigger lab: Basic server-side template injection using ERB / Ruby.
- Review root cause, remediacji i testow regresji.

## Pliki

- [01-overview.md](01-overview.md) - definicja, interpreter, impact i podstawowe rozroznienia.
- [02-template-engines-and-evaluation.md](02-template-engines-and-evaluation.md) - Smarty, Pug, Jinja2, ERB i lekcje z process API.
- [03-detection-and-fingerprinting.md](03-detection-and-fingerprinting.md) - reflection, ewaluacja, bledy i identyfikacja silnika.
- [04-remediation.md](04-remediation.md) - secure design i defence-in-depth.
- [tools.md](tools.md) - SSTImap i manual-first workflow.
- [SOURCES.md](SOURCES.md) - glowne referencje uzyte do weryfikacji tych notatek SSTI.
- [cheat-sheet.md](cheat-sheet.md) - praktyczne przypomnienie do autoryzowanych testow.
- [regression-tests.md](regression-tests.md) - testy odrozniajace odbicie tekstu od ewaluacji szablonu.
- [learning-summary.md](learning-summary.md) - proces nauki, pomylki i koncowe wnioski.
- [labs/01-basic-ssti-erb.md](labs/01-basic-ssti-erb.md) - ukonczony lab PortSwigger.
- [labs/summary.md](labs/summary.md) - porownanie wykonanej praktyki.

## Aktualny status

**PASS**

Mam praktyczna podstawe wystarczajaca dla Frontend Engineera przechodzacego w AppSec. Potrafie wskazac kontrolowany input, odroznic reflection od server-side evaluation, opisac root cause i podac glowny secure design.

Jeden lab PortSwigger jest wystarczajacy na obecnym etapie, bo uzupelnia trzy praktyczne cwiczenia TryHackMe dla roznych silnikow. Kolejne laby SSTI mozna dodac pozniej jako powtorke albo glebsza specjalizacje.

## Kluczowy wniosek

Glowna poprawka nie polega na wiekszej blackliscie znakow szablonu.

```text
Trzymaj template source jako statyczne i kontrolowane przez developerow.
Input uzytkownika przekazuj tylko jako template data.
```
