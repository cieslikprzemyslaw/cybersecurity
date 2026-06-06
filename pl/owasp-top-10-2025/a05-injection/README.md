# A05: Injection

Ten katalog zawiera moje notatki do **OWASP Top 10 2025 - A05: Injection**.

## Zacznij tutaj

- [Overview](01-overview.md)
- [Laby i praktyka](02-labs-or-practice.md)
- [Checklista](03-checklist.md)
- [Pomysły na testy regresji](04-regression-tests.md)
- [Learning notes](05-learning-notes.md)

## Zakres tematu

Ta sekcja obecnie obejmuje:

- [SQL Injection](sql-injection/README.md)
- [NoSQL Injection](nosql-injection/README.md)
- [OS Command Injection](os-command-injection/README.md)

Kolejne tematy A05, takie jak Server-Side Template Injection, AI Prompt Injection i mapowanie XSS, powinny zostać dodane jako osobne moduły dopiero po wykonaniu teorii, labów, review i debriefu.

## Bezpośrednie linki do labów

### SQL Injection

- [UNION-based SQL Injection - retrieving data from other tables](sql-injection/labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](sql-injection/labs/02-blind-conditional-responses.md)

### NoSQL Injection

- [Detecting NoSQL injection](nosql-injection/labs/01-detecting-nosql-injection.md)
- [Exploiting NoSQL injection to extract data](nosql-injection/labs/02-extracting-data-with-a-boolean-oracle.md)
- [Porównanie labów NoSQL](nosql-injection/labs/summary.md)

### OS Command Injection

- [Simple OS command injection](os-command-injection/labs/01-simple-command-injection.md)
- [Blind OS command injection z output redirection](os-command-injection/labs/02-blind-command-injection-output-redirection.md)
- [Porównanie labów OS Command Injection](os-command-injection/labs/summary.md)

## Rola plików

- `01-overview.md` wyjaśnia szerszy model injection i typowe przyczyny źródłowe.
- `02-labs-or-practice.md` zapisuje ukończoną legalną praktykę i aktualne ograniczenia.
- `03-checklist.md` jest praktyczną checklistą do review i testów.
- `04-regression-tests.md` zbiera pomysły na weryfikację poprawek.
- `05-learning-notes.md` jest krótkim indeksem nauki i przekrojowym podsumowaniem.
- `learning-notes/` zawiera dłuższe lekcje i korekty tematyczne.
- `security-findings/` zawiera przykładowe opisy znalezisk oparte na autoryzowanych labach.

## Przykładowe findingi

- [UNION-based SQL Injection w filtrze kategorii](security-findings/01-example-finding-union-based-sqli.md)
- [Boolean-based blind SQL Injection w cookie TrackingId](security-findings/02-example-finding-blind-sqli.md)
- [NoSQL Syntax Injection w filtrze produktów](security-findings/03-example-finding-nosql-syntax-injection-detection.md)
- [NoSQL Syntax Injection pozwalające na blind extraction danych logowania](security-findings/04-example-finding-nosql-blind-extraction.md)
- [Blind OS Command Injection w formularzu feedbacku](security-findings/05-example-finding-blind-os-command-injection.md)
