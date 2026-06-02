# A05: Injection

Ten katalog zawiera moje notatki do **OWASP Top 10 2025 - A05: Injection**.

Główny model mentalny tej kategorii:

```text
Niezaufane dane wejściowe trafiają do interpretera i zmieniają znaczenie zapytania, komendy, szablonu, wyrażenia, promptu albo operacji backendowej.
```

W tej sekcji zaczynam od SQL Injection, bo to jeden z najczytelniejszych przykładów tego modelu:

```text
dane użytkownika -> zapytanie SQL -> interpreter bazy danych -> zmienione zachowanie zapytania
```

## Ukończone do tej pory

- teoria SQL Injection,
- typy SQLi,
- lab UNION-based SQL Injection,
- lab boolean-based blind SQL Injection,
- notatki o encodingu i obchodzeniu filtrów SQLi,
- notatki świadomościowe o out-of-band SQLi,
- przykładowe opisy znalezisk SQL Injection.

## Aktualna struktura

```text
a05-injection/
  README.md
  01-overview.md
  02-labs-or-practice.md
  03-checklist.md
  04-regression-tests.md
  05-learning-notes.md

  sql-injection/
    README.md
    01-overview.md
    02-types-of-sqli.md
    03-in-band-sqli.md
    04-union-based-sqli.md
    05-blind-sqli.md
    06-out-of-band-sqli.md
    07-filter-bypass-and-encoding.md
    08-remediation.md
    cheat-sheet.md
    regression-tests.md
    learning-summary.md

    labs/
      01-union-retrieve-data-from-other-tables.md
      02-blind-conditional-responses.md

  security-findings/
    01-example-finding-union-based-sqli.md
    02-example-finding-blind-sqli.md
```

Kolejne tematy A05, takie jak NoSQL Injection, OS Command Injection, SSTI, AI Prompt Injection i mapowanie XSS, zostaną dodane jako osobne sekcje po wykonaniu odpowiednich labów lub zadań review.
