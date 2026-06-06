# OS Command Injection Overview

## Definition

OS Command Injection occurs when software uses externally influenced input to construct or control an operating-system command and the input changes the intended execution.

It is often called shell injection, although the wider review should also consider unsafe process arguments when no shell is involved.

## Core mental model

```text
controlled input
-> backend command or argument construction
-> shell or process execution API
-> changed execution
-> observable evidence or impact
```

## Controlled input

Potential sources include:

- query parameters,
- form fields,
- JSON body properties,
- route parameters,
- headers,
- cookies,
- filenames,
- uploaded metadata,
- webhook settings,
- CMS values,
- scheduled-job configuration.

A value is not dangerous simply because it is user-controlled. The risk appears when it reaches a security-sensitive sink and can influence command syntax, the executable, flags, arguments, environment, working directory, redirection, or file paths.

## Common process-execution features

Command execution may appear in:

- network diagnostics such as ping or traceroute,
- image and document conversion,
- archive extraction or creation,
- video processing,
- PDF generation,
- email delivery,
- backup and restore jobs,
- antivirus or scanning wrappers,
- Git or deployment tools,
- printer or device integrations,
- legacy scripts,
- administrative and support endpoints.

A production application would usually prefer a library, SDK, or service API over shelling out to an operating-system utility.

## SQL Injection versus OS Command Injection

| Area | SQL Injection | OS Command Injection |
|---|---|---|
| Interpreter | database query engine | shell, command interpreter, or process API |
| Intended operation | database query | operating-system process execution |
| Changed behaviour | query logic or returned records | executed program, arguments, redirection, filesystem or network side effect |
| Typical evidence | database error, returned rows, boolean oracle, delay | direct output, delay, file side effect, outbound interaction |
| Main secure pattern | parameterised queries | avoid OS commands; otherwise fixed executable and separate arguments without a shell |

Both share the same wider Injection failure:

```text
data crosses a boundary and becomes executable instruction
```

## Shell versus process execution

A shell interprets syntax such as separators, pipes, substitutions, quoting, redirection, variables, and wildcards.

Examples include:

- `/bin/sh`,
- Bash,
- `cmd.exe`,
- PowerShell.

A process API can also start an executable directly without using a shell. In that case, shell separators may remain ordinary argument characters. However, unsafe arguments can still change the behaviour of the selected program. This is called argument injection.

## Root cause

A strong root-cause statement is:

> Untrusted input was allowed to influence executable command syntax or security-sensitive process arguments because the application did not maintain separation between data and execution control.

Weak root-cause statements include:

- "The payload contained a semicolon."
- "The user entered bad data."
- "Validation was missing."

Those statements describe symptoms or missing controls, not the unsafe design boundary.

## Impact

Impact depends on the application account and environment. Possible outcomes include:

- disclosure of application-readable files,
- modification of writable files,
- access to configuration or secrets,
- use of installed utilities,
- outbound network access,
- denial of service,
- application takeover,
- movement toward other internal systems if trust and network access allow it.

Least privilege can reduce impact but does not remove the vulnerability.

## Evidence model

Evidence should show that controlled input changed operating-system execution.

Strong examples:

- recognisable command output in the HTTP response,
- repeatable and proportional time delay,
- a predictable file created by redirected output,
- an authorised DNS or HTTP callback,
- an intended application action replaced or supplemented by a command side effect.

Weak examples:

- one slow request,
- a generic `500` response,
- an input-validation error,
- a changed response with no connection to process execution,
- a failed payload that is assumed to identify the operating system.

## Interview-style takeaway

OS Command Injection is a data-flow and execution-boundary problem. The important skill is not memorising separators. It is tracing controlled input to a command or process sink, understanding whether a shell interprets it, producing reliable evidence, and recommending an implementation that removes the shell boundary where possible.
