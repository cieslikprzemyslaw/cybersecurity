# Tools

## SSTImap

Oficjalne repozytorium:

- https://github.com/vladko312/SSTImap

SSTImap to automatyczne narzedzie do wykrywania i oceny Server-Side Template Injection oraz powiazanych zachowan code-injection w wielu template engines.

Potencjalne uzycia w autoryzowanych srodowiskach:

- znajdowanie injectable parameters,
- identyfikowanie prawdopodobnych template engines,
- testowanie zachowan render, error-based i blind,
- ocena mozliwosci takich jak code evaluation albo file access.

## Manual-first rule

Uzywaj SSTImap po zbudowaniu i przetestowaniu recznej hipotezy.

```text
identify feature
  -> identify controlled input
  -> prove reflection
  -> manually test evaluation
  -> form engine hypothesis
  -> use SSTImap as supporting verification
  -> manually confirm important findings
```

Nie traktuj kazdej linii narzedzia jako niepodwazalnego faktu. Zweryfikuj:

- injection point,
- engine identification,
- response evidence,
- zalozenia o systemie operacyjnym,
- raportowane capabilities,
- czy side effect rzeczywiscie wystapil.

## Swiadomosc wersji

Opcje command-line moga zmieniac sie miedzy wersjami. Zawsze sprawdz zainstalowana wersje:

```bash
python3 sstimap.py -h
```

Komenda skopiowana z TryHackMe AttackBox moze miec inne flagi niz najnowsza wersja z GitHuba.

## Scope i safety

Uzywaj SSTImap tylko wobec:

- TryHackMe,
- PortSwigger Web Security Academy,
- lokalnych labow,
- jawnie autoryzowanych celow.

Nie uzywaj automatyzacji do testowania realnych systemow bez pozwolenia.

## Czego narzedzie nie powinno zastepowac

SSTImap nie zastepuje umiejetnosci wyjasnienia:

- jaki input jest kontrolowany,
- gdzie jest renderowany,
- jaki dowod potwierdza ewaluacje,
- dlaczego podejrzewasz konkretny engine,
- jaki jest root cause,
- jak naprawic problem.
