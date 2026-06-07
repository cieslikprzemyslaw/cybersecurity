# A05: Injection - Overview

## What is Injection?

Injection happens when untrusted input is sent to an interpreter or processing engine and changes the meaning of a query, command, template, expression, prompt, or backend operation.

The important point is not the payload itself. The important point is that input controlled by a user is treated as part of something executable or interpretable.

## Core mental model

```text
untrusted input
  -> reaches an interpreter or execution mechanism
  -> is treated as syntax, an operator, an instruction, or a dangerous argument
  -> changes the intended behaviour
```

Examples of interpreters or processing engines include:

- SQL database engine,
- operating-system shell or process execution API,
- NoSQL query engine,
- template engine,
- browser or JavaScript engine,
- LLM or agent instruction-following system.

## Current practical examples

SQL Injection, NoSQL Injection, OS Command Injection, and Server-Side Template Injection are the completed practical modules in this category. Cross-Site Scripting and AI Prompt Injection are documented here as related A05 mappings.

The detailed notes live in:

- [SQL Injection](sql-injection/README.md)
- [NoSQL Injection](nosql-injection/README.md)
- [OS Command Injection](os-command-injection/README.md)
- [Server-Side Template Injection](server-side-template-injection/README.md)
- [XSS Injection Mapping](xss-injection-mapping/README.md)
- [AI Prompt Injection Awareness](ai-prompt-injection-awareness/README.md)

This overview keeps the wider A05 model separate from database-specific, shell-specific, template-specific, and lab-specific payload details.

## Comparing the completed modules

| Topic | Controlled input changes | Main interpreter or mechanism | Typical evidence |
|---|---|---|---|
| SQL Injection | SQL query structure or logic | relational database engine | errors, returned rows, boolean or timing oracle |
| NoSQL Injection | query objects, operators, or custom expressions | NoSQL query engine or embedded expression evaluator | changed records, auth bypass, boolean oracle |
| OS Command Injection | command syntax, executable behaviour, or process arguments | shell, command interpreter, or process API | direct output, timing, file side effect, OAST interaction |
| Server-Side Template Injection | template source, expressions, or rendering logic | server-side template engine | calculated expression result, engine error, information disclosure, file or process side effect |
| Cross-Site Scripting | browser output or DOM execution context | browser / JavaScript engine | script execution, DOM change, browser-side side effect |
| AI Prompt Injection | instruction-following context or tool intent | LLM, agent, retrieval pipeline, or tool router | changed model behaviour, unsafe tool proposal, data exposure, verified action |

## Key AppSec distinction

Injection is not only about finding a working payload. For AppSec, the important questions are:

- What input do I control?
- Where does that input go?
- What interpreter or execution mechanism processes it?
- Is the input still data, or has it become syntax, an operator, an instruction, or a program option?
- What behaviour changed?
- What evidence proves the issue?
- Which observations are facts and which are assumptions?
- What is the root cause?
- What would a secure implementation do differently?
- What regression tests should prevent recurrence?

## Wider secure-design lesson

The secure pattern depends on the interpreter:

- SQL: use parameterised queries.
- NoSQL: enforce schemas, primitive types, and server-selected operators.
- OS commands: avoid command execution where possible; otherwise avoid the shell and pass a fixed executable with separate validated arguments.
- SSTI: keep templates static and developer-controlled; pass user input only as data and never dynamically compile it as template source.
- XSS: use context-aware output encoding, safe DOM APIs, maintained HTML sanitisation where raw HTML is required, and CSP as defence in depth.
- Prompt Injection: separate trusted instructions from untrusted content, enforce authorization and tool permissions outside the model, and treat model output as untrusted.

Validation is useful, but it should not be the only boundary between untrusted input and executable behaviour.

## Current focus

This section currently contains completed notes for SQL Injection, NoSQL Injection, OS Command Injection, and Server-Side Template Injection. Other A05 injection topics will be added only after they are studied, tested, reviewed, and debriefed.
