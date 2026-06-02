# A05: Injection - Overview

## What is Injection?

Injection happens when untrusted input is sent to an interpreter or processing engine and changes the meaning of a query, command, template, expression, prompt, or backend operation.

The important point is not the payload itself. The important point is that input controlled by a user is treated as part of something executable or interpretable.

## Core mental model

```text
untrusted input
  -> reaches an interpreter
  -> is treated as syntax/instruction
  -> changes the intended behaviour
```

Examples of interpreters or processing engines include:

- SQL database engine
- operating system shell
- NoSQL query engine
- template engine
- browser / JavaScript engine
- LLM or agent instruction-following system

## Current practical example

SQL Injection is the first completed practical module in this category.

The detailed SQLi notes live in [sql-injection/](sql-injection/README.md). This overview keeps the wider A05 model separate from SQL-specific payloads and lab details.

## Key AppSec distinction

Injection is not only about finding a working payload. For AppSec, the important questions are:

- What input do I control?
- Where does that input go?
- What interpreter processes it?
- What behaviour changed?
- What evidence proves the issue?
- What is the root cause?
- What would a secure implementation do differently?

## Current focus

This section currently contains completed notes for SQL Injection only. Other A05 injection topics will be added after they are studied and tested.
