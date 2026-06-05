# A05: Injection - Overview

## Czym jest Injection?

Injection występuje wtedy, gdy niezaufane dane wejściowe trafiają do interpretera lub silnika przetwarzającego i zmieniają znaczenie zapytania, komendy, szablonu, wyrażenia, promptu albo operacji backendowej.

Najważniejszy nie jest sam payload. Najważniejsze jest to, że dane kontrolowane przez użytkownika zostają potraktowane jak składnia, instrukcja albo część wykonywalnej operacji.

## Główny model mentalny

```text
niezaufane dane wejściowe
  -> trafiają do interpretera
  -> są potraktowane jak składnia/instrukcja
  -> zmieniają zamierzone zachowanie
```

Przykłady interpreterów lub silników przetwarzających:

- silnik bazy SQL,
- powłoka systemu operacyjnego,
- silnik zapytań NoSQL,
- template engine,
- przeglądarka / silnik JavaScript,
- system LLM lub agent wykonujący instrukcje.

## Aktualne przykłady praktyczne

SQL Injection i NoSQL Injection są ukończonymi modułami praktycznymi w tej kategorii.

Szczegółowe notatki znajdują się w:

- [SQL Injection](sql-injection/README.md)
- [NoSQL Injection](nosql-injection/README.md)

Ten overview zostaje przy szerszym modelu A05, zamiast powtarzać payloady i szczegóły labów zależne od konkretnej bazy.

## Ważne rozróżnienie AppSec

Injection to nie tylko znalezienie działającego payloadu. W AppSec ważne są pytania:

- Jakie dane wejściowe kontroluję?
- Dokąd trafiają te dane?
- Jaki interpreter je przetwarza?
- Jakie zachowanie się zmieniło?
- Jakie dowody potwierdzają problem?
- Jaka jest przyczyna źródłowa?
- Jak wyglądałaby bezpieczna implementacja?

## Aktualny zakres

Ta sekcja zawiera obecnie ukończone notatki dla SQL Injection i NoSQL Injection. Inne tematy A05 zostaną dodane po ich przećwiczeniu i opisaniu.
