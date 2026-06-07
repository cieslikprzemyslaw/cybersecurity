# A05: Injection - Labs and Practice

This category currently contains completed practical work for SQL Injection, NoSQL Injection, OS Command Injection, and Server-Side Template Injection.

All practical testing documented here was performed only in TryHackMe, PortSwigger Web Security Academy, local labs, or another explicitly authorised environment.

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

## Completed OS Command Injection practice

### TryHackMe

- OS Command Injection theory
- PHP `exec()` example using a user-controlled song title
- Python Flask example using `subprocess.Popen(..., shell=True)`
- Direct/verbose versus blind command injection
- Timing, output redirection, and high-level out-of-band evidence
- Linux and Windows command differences
- Defensive priorities and the limits of sanitisation

### PortSwigger Web Security Academy

- [Simple OS command injection](os-command-injection/labs/01-simple-command-injection.md)
- [Blind OS command injection with output redirection](os-command-injection/labs/02-blind-command-injection-output-redirection.md)
- [OS Command Injection lab comparison](os-command-injection/labs/summary.md)

## Completed Server-Side Template Injection practice

### TryHackMe

- SSTI theory and engine-specific practical work
- Smarty / PHP template evaluation and security policy awareness
- Pug / Node.js expression evaluation and process API lessons
- Jinja2 / Python object traversal and subprocess API lessons
- SSTImap awareness for authorised lab use

### PortSwigger Web Security Academy

- [Basic server-side template injection using ERB](server-side-template-injection/labs/01-basic-ssti-erb.md)
- [SSTI practical work comparison](server-side-template-injection/labs/summary.md)

## Practice coverage

The completed practice covers:

- identifying user-controlled input that reaches an interpreter or execution mechanism,
- distinguishing data from executable syntax,
- recognising when a backend may build a full shell command,
- separating facts from assumptions about the original backend command,
- establishing a normal baseline before interpreting response changes,
- using direct command output as evidence,
- using repeatable and proportional timing as blind evidence,
- using a file side effect and a separate retrieval endpoint to recover hidden output,
- understanding why a `500` response does not automatically prove that execution failed,
- explaining why a closing separator may be required to isolate an injected command,
- distinguishing shell injection from argument injection,
- distinguishing reflected text from server-side template evaluation,
- fingerprinting a template engine with harmless, engine-specific evidence,
- separating SSTI root cause from deeper impacts such as file access or process execution,
- translating lab evidence into root cause, remediation, and regression tests.

## Current limits

This folder does not yet contain completed practical modules for:

- AI Prompt Injection,
- XSS mapping under A05.

Those topics should be added only after the relevant learning and practical work are completed.
