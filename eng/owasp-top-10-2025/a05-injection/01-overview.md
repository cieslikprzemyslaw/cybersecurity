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

SQL Injection, NoSQL Injection, and OS Command Injection are the completed practical modules in this category.

The detailed notes live in:

- [SQL Injection](sql-injection/README.md)
- [NoSQL Injection](nosql-injection/README.md)
- [OS Command Injection](os-command-injection/README.md)

This overview keeps the wider A05 model separate from database-specific, shell-specific, and lab-specific payload details.

## Comparing the completed modules

| Topic | Controlled input changes | Main interpreter or mechanism | Typical evidence |
|---|---|---|---|
| SQL Injection | SQL query structure or logic | relational database engine | errors, returned rows, boolean or timing oracle |
| NoSQL Injection | query objects, operators, or custom expressions | NoSQL query engine or embedded expression evaluator | changed records, auth bypass, boolean oracle |
| OS Command Injection | command syntax, executable behaviour, or process arguments | shell, command interpreter, or process API | direct output, timing, file side effect, OAST interaction |

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

Validation is useful, but it should not be the only boundary between untrusted input and executable behaviour.

## Current focus

This section currently contains completed notes for SQL Injection, NoSQL Injection, and OS Command Injection. Other A05 injection topics will be added only after they are studied, tested, reviewed, and debriefed.
