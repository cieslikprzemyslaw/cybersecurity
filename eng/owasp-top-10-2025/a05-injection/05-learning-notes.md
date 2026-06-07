# A05: Injection - Learning Notes

This file is the short index for A05 learning notes. Longer topic-specific reflections live in `learning-notes/` so this page stays readable.

## Wider mental model

SQL Injection, NoSQL Injection, OS Command Injection, and Server-Side Template Injection all reinforce the wider A05 model:

```text
controlled input -> interpreter or execution mechanism -> changed behaviour
```

The payload is not the root cause by itself. The vulnerability exists because the application allows untrusted input to become query syntax, a query operator, template syntax, part of an executable expression, shell syntax, or a dangerous process argument.

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
- Browser-side script execution and model-generated instructions both need realistic impact analysis rather than inflated claims.
- Model output is not the same as verified tool execution, data access, or state change.
- Root cause should describe the failed data/execution boundary, not only the payload.
- Remediation should remove unsafe interpreter boundaries where possible, then add validation, least privilege, monitoring, and regression tests.

## Current understanding

I should be able to explain:

- what input I control,
- which endpoint actually processes it,
- whether the input is a value, object, operator, shell syntax, or process argument,
- what parser, query engine, shell, process API, or template engine handles it,
- what evidence proves interpreter or execution behaviour changed,
- how an oracle or side effect can reveal hidden behaviour,
- what is fact and what is assumption,
- what the root cause is,
- how a developer should fix it,
- what regression tests should prevent recurrence.

I do not need to memorise every operator or payload. I need to understand data flow, execution context, evidence, impact, remediation, and verification.
