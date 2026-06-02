# A05: Injection

This folder contains my learning notes for **OWASP Top 10 2025 - A05: Injection**.

The main mental model for this category is:

```text
Untrusted input reaches an interpreter and changes the meaning of a query, command, template, expression, prompt, or backend operation.
```

For this section, I started with SQL Injection because it is one of the clearest examples of this model:

```text
user input -> SQL query -> database interpreter -> changed query behaviour
```

## Completed so far

- SQL Injection theory
- SQL Injection types
- UNION-based SQL Injection lab
- Boolean-based blind SQL Injection lab
- SQLi filter bypass and encoding notes
- Out-of-band SQL Injection awareness notes
- SQL Injection security finding examples

## Current structure

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

Future A05 topics such as NoSQL Injection, OS Command Injection, SSTI, AI Prompt Injection, and XSS mapping will be added later as separate sections after I complete the relevant learning and labs.
