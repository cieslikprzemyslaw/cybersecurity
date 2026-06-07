# A05: Injection - Overview

## Czym jest Injection?

Injection występuje wtedy, gdy niezaufane dane wejściowe trafiają do interpretera lub silnika przetwarzającego i zmieniają znaczenie zapytania, komendy, szablonu, wyrażenia, promptu albo operacji backendowej.

Najważniejszy nie jest sam payload. Najważniejsze jest to, że dane kontrolowane przez użytkownika zostają potraktowane jak składnia, instrukcja albo część wykonywalnej operacji.

## Główny model mentalny

```text
niezaufane dane wejściowe
  -> trafiają do interpretera albo mechanizmu wykonania
  -> są potraktowane jak składnia, operator, instrukcja albo niebezpieczny argument
  -> zmieniają zamierzone zachowanie
```

Przykłady interpreterów lub silników przetwarzających:

- silnik bazy SQL,
- powłoka systemu operacyjnego albo API uruchamiania procesów,
- silnik zapytań NoSQL,
- template engine,
- przeglądarka / silnik JavaScript,
- system LLM lub agent wykonujący instrukcje.

## Aktualne przykłady praktyczne

SQL Injection, NoSQL Injection, OS Command Injection i Server-Side Template Injection są ukończonymi modułami praktycznymi w tej kategorii. Cross-Site Scripting i AI Prompt Injection są opisane tutaj jako powiązane mapowania A05.

Szczegółowe notatki znajdują się w:

- [SQL Injection](sql-injection/README.md)
- [NoSQL Injection](nosql-injection/README.md)
- [OS Command Injection](os-command-injection/README.md)
- [Server-Side Template Injection](server-side-template-injection/README.md)
- [XSS Injection Mapping](xss-injection-mapping/README.md)
- [AI Prompt Injection Awareness](ai-prompt-injection-awareness/README.md)

Ten overview zostaje przy szerszym modelu A05, zamiast powtarzać payloady i szczegóły labów zależne od konkretnej bazy, powłoki, template engine albo środowiska.

## Porównanie ukończonych modułów

| Temat | Kontrolowany input zmienia | Główny interpreter albo mechanizm | Typowe dowody |
|---|---|---|---|
| SQL Injection | strukturę lub logikę zapytania SQL | relacyjny silnik bazy danych | błędy, zwrócone wiersze, boolean oracle, timing |
| NoSQL Injection | obiekty query, operatory lub custom expressions | silnik NoSQL albo osadzony evaluator wyrażeń | zmienione rekordy, auth bypass, boolean oracle |
| OS Command Injection | składnię komendy, zachowanie programu lub argumenty procesu | shell, command interpreter albo process API | output w odpowiedzi, timing, side effect w pliku, OAST |
| Server-Side Template Injection | źródło szablonu, wyrażenia albo logikę renderowania | server-side template engine | obliczony wynik wyrażenia, błąd silnika, information disclosure, side effect w pliku albo procesie |
| Cross-Site Scripting | output przeglądarki albo DOM execution context | przeglądarka / JavaScript engine | script execution, zmiana DOM, browser-side side effect |
| AI Prompt Injection | instruction-following context albo intencję tool use | LLM, agent, retrieval pipeline albo tool router | zmiana zachowania modelu, unsafe tool proposal, data exposure, verified action |

## Ważne rozróżnienie AppSec

Injection to nie tylko znalezienie działającego payloadu. W AppSec ważne są pytania:

- Jakie dane wejściowe kontroluję?
- Dokąd trafiają te dane?
- Jaki interpreter albo mechanizm wykonania je przetwarza?
- Czy input nadal jest danymi, czy stał się składnią, operatorem, instrukcją albo opcją programu?
- Jakie zachowanie się zmieniło?
- Jakie dowody potwierdzają problem?
- Jaka jest przyczyna źródłowa?
- Jak wyglądałaby bezpieczna implementacja?
- Jakie testy regresji powinny zapobiec powrotowi problemu?

## Szersza lekcja projektowa

Bezpieczny wzorzec zależy od interpretera:

- SQL: używaj zapytań parametryzowanych.
- NoSQL: egzekwuj schematy, typy prymitywne i operatory wybierane po stronie serwera.
- OS commands: unikaj uruchamiania komend, jeśli można użyć biblioteki lub API; jeśli proces jest konieczny, użyj stałego executable i osobnych zwalidowanych argumentów bez shella.
- SSTI: trzymaj szablony jako statyczne i kontrolowane przez developerów; input użytkownika przekazuj tylko jako dane i nigdy nie kompiluj go dynamicznie jako źródła szablonu.
- XSS: używaj context-aware output encoding, safe DOM APIs, maintained HTML sanitisation tam, gdzie raw HTML jest wymagany, oraz CSP jako defence in depth.
- Prompt Injection: oddziel trusted instructions od untrusted content, egzekwuj authorization i tool permissions poza modelem, a model output traktuj jako niezaufany.

Walidacja pomaga, ale nie powinna być jedyną granicą między niezaufanym inputem a wykonywalnym zachowaniem.

## Aktualny zakres

Ta sekcja zawiera obecnie ukończone notatki dla SQL Injection, NoSQL Injection, OS Command Injection i Server-Side Template Injection oraz mapowania A05 dla Cross-Site Scripting i AI Prompt Injection awareness.

Głębsze moduły praktyczne XSS albo LLM-specific powinny zostać dodane dopiero po odpowiedniej nauce, testach, review i debriefie.
