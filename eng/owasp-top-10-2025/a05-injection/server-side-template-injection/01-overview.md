# SSTI Overview

## What is a server-side template engine?

A server-side template engine combines a developer-controlled template with data to generate output such as HTML, emails, documents, or configuration text.

Example concept:

```text
static template:  Hello, {{ name }}
data:             name = "Przemyslaw"
rendered output:  Hello, Przemyslaw
```

Applications use templates to avoid manually building every page and to separate presentation from application logic.

## What is SSTI?

SSTI occurs when untrusted input is included in template source and the engine interprets that input as template syntax or an expression.

```text
safe:
static template + user input as data

unsafe:
template source constructed from user input + compile/render
```

## Evidence levels

### 1. Plain reflection

The input appears in the response exactly as supplied.

```text
input:  SSTI_TEST_123
output: SSTI_TEST_123
```

This proves control of the input and its output location. It does not prove SSTI.

### 2. HTML rendering

The browser interprets generated markup. This may be relevant to XSS, but it does not by itself prove a server-side template engine evaluated an expression.

### 3. Template expression evaluation

A template expression is replaced by its calculated result.

```text
input:  engine-specific expression for 7 * 7
output: 49
```

This is evidence that a server-side template engine evaluated the expression.

### 4. Deeper impact

Further testing may show access to:

- template context variables,
- application objects,
- environment variables,
- files,
- language runtime APIs,
- process execution APIs.

Expression evaluation does not automatically prove remote code execution. Each deeper capability requires separate evidence.

## SSTI versus XSS

| Question | SSTI | XSS |
|---|---|---|
| Main interpreter | Server-side template engine | Browser / JavaScript engine |
| Processing time | Before the response is generated | After the response reaches the browser |
| Typical evidence | Expression evaluated into a result | Script or browser behaviour executes |
| Main target | Server application and its privileges | User session and browser context |

HTML escaping may help prevent XSS in generated output, but it does not fix user input becoming executable template source.

## SSTI versus OS Command Injection

| Question | SSTI | OS Command Injection |
|---|---|---|
| Initial root cause | Input changes template evaluation | Input changes a shell command or process invocation |
| Initial interpreter | Template engine | Shell, command interpreter, or unsafe process API use |
| Possible escalation | Language/runtime access may lead to process execution | Direct command execution is already the vulnerable behaviour |

SSTI can lead to OS command execution, but command execution is an impact path. It does not change the original root cause.

## Potential impact

Depending on the engine, runtime, context, configuration, and application privileges, impact may include:

- changed server-generated content,
- information disclosure,
- access to application configuration or secrets,
- file read, write, or deletion,
- server-side code execution,
- operating-system process execution,
- compromise of data available to the application process.

Impact is limited by the privileges and environment of the application process. SSTI does not automatically mean root or administrator access.

## Root cause

The root cause is a data/code boundary failure:

> User-controlled content was treated as template source instead of being passed as an ordinary variable to a static template.

This is more precise than saying only "the input was not sanitised".
