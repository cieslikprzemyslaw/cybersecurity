# A05: Injection

This folder contains my learning notes for OWASP Top 10 2025 - A05: Injection.

## Start Here

- [Overview](01-overview.md)
- [Labs and practice](02-labs-or-practice.md)
- [Checklist](03-checklist.md)
- [Regression test ideas](04-regression-tests.md)
- [Learning notes](05-learning-notes.md)

## Topic Scope

This section currently covers:

- [SQL Injection](sql-injection/README.md)
- [NoSQL Injection](nosql-injection/README.md)
- [OS Command Injection](os-command-injection/README.md)

Future A05 topics such as Server-Side Template Injection, AI Prompt Injection, and XSS mapping should be added as separate modules only after the relevant theory, labs, review, and debrief are completed.

## Direct Lab Links

### SQL Injection

- [UNION-based SQL Injection - retrieving data from other tables](sql-injection/labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](sql-injection/labs/02-blind-conditional-responses.md)

### NoSQL Injection

- [Detecting NoSQL injection](nosql-injection/labs/01-detecting-nosql-injection.md)
- [Exploiting NoSQL injection to extract data](nosql-injection/labs/02-extracting-data-with-a-boolean-oracle.md)
- [NoSQL lab comparison](nosql-injection/labs/summary.md)

### OS Command Injection

- [Simple OS command injection](os-command-injection/labs/01-simple-command-injection.md)
- [Blind OS command injection with output redirection](os-command-injection/labs/02-blind-command-injection-output-redirection.md)
- [OS Command Injection lab comparison](os-command-injection/labs/summary.md)

## File Roles

- `01-overview.md` explains the wider injection model and common root causes.
- `02-labs-or-practice.md` records completed legal practice and current limits.
- `03-checklist.md` is a practical review and testing checklist.
- `04-regression-tests.md` collects fix-verification ideas.
- `05-learning-notes.md` is a short learning index and cross-topic summary.
- `learning-notes/` contains longer topic-specific lessons and corrections.
- `security-findings/` contains example report write-ups based on authorised labs.

## Example Findings

- [UNION-based SQL Injection in category filter](security-findings/01-example-finding-union-based-sqli.md)
- [Boolean-based blind SQL Injection in TrackingId cookie](security-findings/02-example-finding-blind-sqli.md)
- [NoSQL Syntax Injection in product filter](security-findings/03-example-finding-nosql-syntax-injection-detection.md)
- [NoSQL Syntax Injection allowing blind credential extraction](security-findings/04-example-finding-nosql-blind-extraction.md)
- [Blind OS Command Injection in feedback submission](security-findings/05-example-finding-blind-os-command-injection.md)
