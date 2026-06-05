# A05: Injection - Labs and Practice

This category currently contains practical work for SQL Injection and NoSQL Injection.

## Completed SQL Injection labs

- [UNION-based SQL Injection - retrieving data from other tables](sql-injection/labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](sql-injection/labs/02-blind-conditional-responses.md)

## Completed NoSQL Injection practice

### TryHackMe

- NoSQL Injection room
- MongoDB document and collection fundamentals
- MongoDB query filters and nested operators
- Operator Injection authentication bypass
- `$regex`-based true/false extraction
- Syntax Injection awareness using custom JavaScript-style queries

### PortSwigger Web Security Academy

- [Detecting NoSQL injection](nosql-injection/labs/01-detecting-nosql-injection.md)
- [Exploiting NoSQL injection to extract data](nosql-injection/labs/02-extracting-data-with-a-boolean-oracle.md)
- [NoSQL lab comparison](nosql-injection/labs/summary.md)

## Practice coverage

The completed practice covers:

- identifying user-controlled input that reaches a database query,
- distinguishing a simple value from a nested query object or operator,
- recognising visible injection evidence in HTTP responses,
- using a baseline before interpreting errors or response changes,
- detecting syntax-sensitive query construction with a single quote,
- comparing true and false conditions,
- using response behaviour as a boolean oracle,
- determining a secret length,
- extracting secret characters one position at a time,
- using Burp Intruder only after manually confirming the oracle,
- translating technical evidence into remediation and regression tests.

## Current limits

This folder does not yet contain completed practical modules for:

- OS Command Injection,
- Server-Side Template Injection,
- AI Prompt Injection,
- XSS mapping under A05.

Those topics should be added only after the relevant learning and practical work are completed.
