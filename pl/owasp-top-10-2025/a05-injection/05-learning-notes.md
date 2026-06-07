# A05: Injection - Learning Notes

Ten plik jest krótkim indeksem notatek z nauki A05. Dłuższe refleksje tematyczne są w `learning-notes/`, żeby ta strona pozostała czytelna.

## Szerszy model mentalny

SQL Injection, NoSQL Injection, OS Command Injection i Server-Side Template Injection wzmacniają ten sam szerszy model A05:

```text
kontrolowany input -> interpreter albo mechanizm wykonania -> zmienione zachowanie
```

XSS i Prompt Injection rozszerzają ten model, ale nie sprawiają, że wszystkie klasy injection są takie same. W XSS interpreterem staje się przeglądarka. W Prompt Injection problem przenosi się do LLM albo agentowego kontekstu wykonywania instrukcji.

Payload nie jest sam w sobie przyczyną źródłową. Podatność istnieje, gdy aplikacja pozwala, aby niezaufany input stał się składnią query, operatorem query, składnią szablonu, browser-active content, częścią wykonywalnego wyrażenia, składnią shella, niebezpiecznym argumentem procesu albo instrukcją wpływającą na workflow LLM.

## Szczegółowe notatki

- [Lekcje SQL i NoSQL Injection](learning-notes/sql-and-nosql-injection.md)
- [Lekcje OS Command Injection](learning-notes/os-command-injection.md)
- [Lekcje Server-Side Template Injection](server-side-template-injection/learning-summary.md)
- [AI Prompt Injection awareness](ai-prompt-injection-awareness/README.md)
- [XSS Injection mapping](xss-injection-mapping/README.md)

## Lekcje przekrojowe

- Raw input i encoded input nie są wymienne.
- Encoding zmienia reprezentację transportową; nie sprawia, że wartość jest zaufana.
- Background requests i API calls są ważniejsze niż widoczny URL w przeglądarce.
- Generyczna odpowiedź `500` jest wskazówką, nie dowodem skutecznego wykorzystania.
- Mocny dowód jest konkretny, powtarzalny, kontrolowany i powiązany z podejrzanym interpreterem albo ścieżką wykonania.
- Reflection potwierdza kontrolę inputu; nie potwierdza server-side template evaluation.
- Obliczony wynik wyrażenia szablonu potwierdza ewaluację, ale głębsze możliwości wymagają osobnych dowodów.
- Dowód XSS musi pokazywać browser-side execution albo inny meaningful browser-side effect, nie tylko odbity HTML.
- SSTI i XSS mogą wyglądać podobnie na poziomie payloadu, ale wykonują się w innych miejscach i wymagają innych dowodów.
- Browser-side script execution i model-generated instructions wymagają realistycznej analizy impactu, a nie zawyżonych claimów.
- Model output nie jest tym samym co zweryfikowane tool execution, data access albo state change.
- Prompt Injection najlepiej traktować jako problem kontroli aplikacyjnych: prompty pomagają, ale authorization, retrieval scope, tool permissions, validation i approvals muszą być poza modelem.
- Root cause powinien opisywać nieudaną granicę między danymi a wykonaniem, nie tylko payload.
- Remediacja powinna usuwać niebezpieczne granice interpretera tam, gdzie to możliwe, a dopiero potem dodawać walidację, least privilege, monitoring i testy regresji.

## Co zmieniło się w moim modelu A05

Wcześniej myślałem o injection głównie jako o sytuacji "input trafia do backendowego interpretera". Rozszerzone notatki A05 pokazują, że to za wąskie.

- SQLi i NoSQLi nauczyły mnie śledzić input do struktury query.
- OS Command Injection nauczyło mnie oddzielać składnię shella, wybór executable, argumenty i side effects procesu.
- SSTI nauczyło mnie, że reflection to nie evaluation, a impact zależy od server-side template context.
- XSS nauczył mnie najpierw rozpoznawać finalny browser context i source-to-sink path, a dopiero potem dobierać payload.
- Prompt Injection nauczyło mnie, że model behaviour to nie to samo co verified execution, a bezpieczeństwo LLM zależy od kontroli w aplikacji wokół modelu.

Wspólna umiejętność nadal jest ta sama: wskazać kontrolowany input, wskazać interpreter albo mechanizm decyzji, udowodnić zmianę zachowania, opisać realistyczny impact i zaproponować kontrolę, która usuwa albo ogranicza niebezpieczną granicę.

## Aktualne rozumienie

Powinienem umieć wyjaśnić:

- jaki input kontroluję,
- który endpoint naprawdę go przetwarza,
- czy input jest wartością, obiektem, operatorem, browser-active content, składnią szablonu, składnią shella, argumentem procesu albo instruction-like content,
- jaki parser, query engine, shell, process API, template engine, browser engine albo LLM workflow go przetwarza,
- jaki dowód pokazuje, że zachowanie interpretera albo wykonania się zmieniło,
- jak oracle albo side effect może ujawniać ukryte zachowanie,
- czy dowód potwierdza tylko output, proposed action, czy realne execution,
- co jest faktem, a co założeniem,
- jaka jest przyczyna źródłowa,
- jak developer powinien to naprawić,
- jakie testy regresji powinny zapobiec powrotowi problemu.

Nie muszę pamiętać każdego operatora albo payloadu. Muszę rozumieć data flow, kontekst wykonania, dowody, impact, remediację i weryfikację.
