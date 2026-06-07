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
- [Server-Side Template Injection](server-side-template-injection/README.md)
- [Cross-Site Scripting mapping](xss-injection-mapping/README.md)
- [AI Prompt Injection awareness](ai-prompt-injection-awareness/README.md)

Prompt Injection jest powiązane z ideą injection, ale OWASP dokumentuje pełny temat osobno w OWASP Top 10 for LLM Applications jako LLM01:2025 Prompt Injection. Ten folder A05 zachowuje tylko krótkie mapowanie awareness.

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

### Server-Side Template Injection

- [Basic server-side template injection using ERB](server-side-template-injection/labs/01-basic-ssti-erb.md)
- [Porównanie praktyki SSTI](server-side-template-injection/labs/summary.md)

## Powiązane mapowania Injection

### Cross-Site Scripting

Cross-Site Scripting jest częścią OWASP A05:2025 Injection, bo dane kontrolowane przez atakującego trafiają do interpretera przeglądarki i są traktowane jak aktywna treść.

- [XSS Injection Mapping](xss-injection-mapping/README.md)
- [XSS Testing Cheatsheet](xss-injection-mapping/testing-cheatsheet.md)

### AI Prompt Injection

Prompt Injection jest ryzykiem w stylu injection dla aplikacji LLM. Pełne notatki są w:

```text
pl/owasp-top-10-for-llm-applications-2025/
  llm01-prompt-injection/
```

- [AI Prompt Injection Awareness](ai-prompt-injection-awareness/README.md)

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
- [Server-Side Template Injection w dynamicznym renderowaniu komunikatu](security-findings/06-example-finding-server-side-template-injection.md)
