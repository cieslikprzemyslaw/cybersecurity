# A05: Injection - Learning Notes

This file is the short index for A05 learning notes. Longer topic-specific reflections live in `learning-notes/` so this page stays readable.

## Wider mental model

SQL Injection, NoSQL Injection, OS Command Injection, and Server-Side Template Injection all reinforce the wider A05 model:

```text
controlled input -> interpreter or execution mechanism -> changed behaviour
```

XSS and Prompt Injection widen the same model without making every injection class identical. XSS moves the interpreter to the browser. Prompt Injection moves the problem into an LLM or agent instruction-following context.

The payload is not the root cause by itself. The vulnerability exists because the application allows untrusted input to become query syntax, a query operator, template syntax, browser-active content, part of an executable expression, shell syntax, a dangerous process argument, or an instruction-like influence over an LLM workflow.

## Detailed notes

- [SQL and NoSQL Injection lessons](learning-notes/sql-and-nosql-injection.md)
- [OS Command Injection lessons](learning-notes/os-command-injection.md)
- [Server-Side Template Injection lessons](server-side-template-injection/learning-summary.md)
- [AI Prompt Injection awareness](ai-prompt-injection-awareness/README.md)
- [XSS Injection mapping](xss-injection-mapping/README.md)

## Cross-topic lessons

- Raw input and encoded input are not interchangeable.
- Encoding changes transport representation; it does not make a value trusted.
- Background requests and API calls matter more than the visible browser URL.
- A generic `500` response is a clue, not proof of successful exploitation.
- Strong evidence is specific, repeatable, controlled, and tied to the suspected interpreter or execution path.
- Reflection proves input control; it does not prove server-side template evaluation.
- A calculated template expression proves evaluation, but deeper capabilities require separate evidence.
- XSS evidence must show browser-side execution or another meaningful browser-side effect, not just reflected HTML.
- SSTI and XSS can both involve markup-looking input, but they run in different places and need different evidence.
- Browser-side script execution and model-generated instructions both need realistic impact analysis rather than inflated claims.
- Model output is not the same as verified tool execution, data access, or state change.
- Prompt Injection is best treated as an application-control problem: prompts help, but authorization, retrieval scope, tool permissions, validation, and approvals must live outside the model.
- Root cause should describe the failed data/execution boundary, not only the payload.
- Remediation should remove unsafe interpreter boundaries where possible, then add validation, least privilege, monitoring, and regression tests.

## What changed in my A05 mental model

Earlier, I mostly thought about injection as "input reaches a backend interpreter". The expanded A05 notes make that too narrow.

- SQLi and NoSQLi taught me to trace input into query structure.
- OS Command Injection taught me to separate shell syntax, executable selection, arguments, and process side effects.
- SSTI taught me that reflection is not evaluation, and that server-side template context determines impact.
- XSS taught me to identify the final browser context and source-to-sink path before choosing any payload.
- Prompt Injection taught me that model behaviour is not the same as verified execution, and that LLM security depends on the surrounding application controls.

The shared skill is still the same: identify the controlled input, identify the interpreter or decision mechanism, prove the changed behaviour, explain realistic impact, and recommend the control that removes or constrains the unsafe boundary.

## Current understanding

I should be able to explain:

- what input I control,
- which endpoint actually processes it,
- whether the input is a value, object, operator, browser-active content, template syntax, shell syntax, process argument, or instruction-like content,
- what parser, query engine, shell, process API, template engine, browser engine, or LLM workflow handles it,
- what evidence proves interpreter or execution behaviour changed,
- how an oracle or side effect can reveal hidden behaviour,
- whether the evidence proves only output, proposed action, or actual execution,
- what is fact and what is assumption,
- what the root cause is,
- how a developer should fix it,
- what regression tests should prevent recurrence.

I do not need to memorise every operator or payload. I need to understand data flow, execution context, evidence, impact, remediation, and verification.
