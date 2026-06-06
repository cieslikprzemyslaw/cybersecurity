# A05: Injection - Learning Notes

Ten plik jest krótkim indeksem notatek z nauki A05. Dłuższe refleksje tematyczne są w `learning-notes/`, żeby ta strona pozostała czytelna.

## Szerszy model mentalny

SQL Injection, NoSQL Injection i OS Command Injection wzmacniają ten sam szerszy model A05:

```text
kontrolowany input -> interpreter albo mechanizm wykonania -> zmienione zachowanie
```

Payload nie jest sam w sobie przyczyną źródłową. Podatność istnieje, gdy aplikacja pozwala, aby niezaufany input stał się składnią query, operatorem query, częścią wykonywalnego wyrażenia, składnią shella albo niebezpiecznym argumentem procesu.

## Szczegółowe notatki

- [Lekcje SQL i NoSQL Injection](learning-notes/sql-and-nosql-injection.md)
- [Lekcje OS Command Injection](learning-notes/os-command-injection.md)

## Lekcje przekrojowe

- Raw input i encoded input nie są wymienne.
- Encoding zmienia reprezentację transportową; nie sprawia, że wartość jest zaufana.
- Background requests i API calls są ważniejsze niż widoczny URL w przeglądarce.
- Generyczna odpowiedź `500` jest wskazówką, nie dowodem skutecznego wykorzystania.
- Mocny dowód jest konkretny, powtarzalny, kontrolowany i powiązany z podejrzanym interpreterem albo ścieżką wykonania.
- Root cause powinien opisywać nieudaną granicę między danymi a wykonaniem, nie tylko payload.
- Remediacja powinna usuwać niebezpieczne granice interpretera tam, gdzie to możliwe, a dopiero potem dodawać walidację, least privilege, monitoring i testy regresji.

## Aktualne rozumienie

Powinienem umieć wyjaśnić:

- jaki input kontroluję,
- który endpoint naprawdę go przetwarza,
- czy input jest wartością, obiektem, operatorem, składnią shella albo argumentem procesu,
- jaki parser, query engine, shell albo process API go przetwarza,
- jaki dowód pokazuje, że zachowanie interpretera albo wykonania się zmieniło,
- jak oracle albo side effect może ujawniać ukryte zachowanie,
- co jest faktem, a co założeniem,
- jaka jest przyczyna źródłowa,
- jak developer powinien to naprawić,
- jakie testy regresji powinny zapobiec powrotowi problemu.

Nie muszę pamiętać każdego operatora albo payloadu. Muszę rozumieć data flow, kontekst wykonania, dowody, impact, remediację i weryfikację.
