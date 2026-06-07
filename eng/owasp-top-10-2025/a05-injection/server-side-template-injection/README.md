# Server-Side Template Injection

Server-Side Template Injection (SSTI) happens when untrusted input becomes part of a server-side template source and is evaluated by a template engine.

## Mental model

```text
user-controlled input
  -> server-side template source
  -> template engine
  -> expression evaluation or changed rendering behaviour
  -> possible access to application objects, files, processes, or secrets
```

The key issue is not simply that user input appears in HTML. The issue is that the server treats that input as template syntax instead of ordinary data.

## Completed learning

- TryHackMe theory and practical work with:
  - Smarty / PHP
  - Pug / Node.js
  - Jinja2 / Python
- SSTImap awareness and authorised-lab usage
- PortSwigger lab: Basic server-side template injection using ERB / Ruby
- Review of root cause, remediation, and regression testing

## Files

- [01-overview.md](01-overview.md) - definition, interpreter, impact, and core distinctions.
- [02-template-engines-and-evaluation.md](02-template-engines-and-evaluation.md) - Smarty, Pug, Jinja2, ERB, and process API lessons.
- [03-detection-and-fingerprinting.md](03-detection-and-fingerprinting.md) - reflection, evaluation, errors, and engine identification.
- [04-remediation.md](04-remediation.md) - secure design and defence-in-depth controls.
- [tools.md](tools.md) - SSTImap and manual-first testing workflow.
- [SOURCES.md](SOURCES.md) - primary references used to verify these SSTI notes.
- [cheat-sheet.md](cheat-sheet.md) - practical reminders for authorised testing.
- [regression-tests.md](regression-tests.md) - tests that distinguish text reflection from template evaluation.
- [learning-summary.md](learning-summary.md) - personal learning process, mistakes, and final takeaways.
- [labs/01-basic-ssti-erb.md](labs/01-basic-ssti-erb.md) - completed PortSwigger lab.
- [labs/summary.md](labs/summary.md) - comparison of completed practical work.

## Current status

**PASS**

I have a practical foundation suitable for a Frontend Engineer moving into AppSec. I can identify controlled input, distinguish reflection from server-side evaluation, explain the root cause, and describe the primary secure design.

One PortSwigger lab is sufficient for the current stage because it follows three TryHackMe engine-specific practical exercises. More SSTI labs can be added later as revision or deeper specialisation rather than as a requirement before continuing A05.

## Key takeaway

The primary fix is not a larger blacklist of template characters.

```text
Keep template source static and developer-controlled.
Pass user input only as template data.
```
