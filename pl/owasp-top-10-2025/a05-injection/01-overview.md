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

SQL Injection, NoSQL Injection i OS Command Injection są ukończonymi modułami praktycznymi w tej kategorii.

Szczegółowe notatki znajdują się w:

- [SQL Injection](sql-injection/README.md)
- [NoSQL Injection](nosql-injection/README.md)
- [OS Command Injection](os-command-injection/README.md)

Ten overview zostaje przy szerszym modelu A05, zamiast powtarzać payloady i szczegóły labów zależne od konkretnej bazy, powłoki albo środowiska.

## Porównanie ukończonych modułów

| Temat | Kontrolowany input zmienia | Główny interpreter albo mechanizm | Typowe dowody |
|---|---|---|---|
| SQL Injection | strukturę lub logikę zapytania SQL | relacyjny silnik bazy danych | błędy, zwrócone wiersze, boolean oracle, timing |
| NoSQL Injection | obiekty query, operatory lub custom expressions | silnik NoSQL albo osadzony evaluator wyrażeń | zmienione rekordy, auth bypass, boolean oracle |
| OS Command Injection | składnię komendy, zachowanie programu lub argumenty procesu | shell, command interpreter albo process API | output w odpowiedzi, timing, side effect w pliku, OAST |

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

Walidacja pomaga, ale nie powinna być jedyną granicą między niezaufanym inputem a wykonywalnym zachowaniem.

## Aktualny zakres

Ta sekcja zawiera obecnie ukończone notatki dla SQL Injection, NoSQL Injection i OS Command Injection. Inne tematy A05 zostaną dodane po ich przećwiczeniu, review i debriefie.
