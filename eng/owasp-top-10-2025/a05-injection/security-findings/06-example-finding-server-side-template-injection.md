# Example Finding: Server-Side Template Injection in Dynamic Message Rendering

## Summary

A user-controlled `message` parameter is incorporated into ERB template source and evaluated on the server. An attacker can inject Ruby template expressions and access dangerous server-side APIs available to the application process.

## Severity

**High**

Severity depends on the objects, APIs, secrets, filesystem paths, and operating-system privileges available to the template context and application process.

## Affected feature

Dynamic out-of-stock message rendering.

## Evidence

A unique marker supplied through `message` was reflected in the generated HTML.

A harmless ERB arithmetic expression was then evaluated and replaced with its calculated result, confirming that the value was interpreted as template syntax rather than ordinary text.

In the authorised lab, the same injection point reached Ruby's `File` API and deleted the file specified by the lab objective.

## Impact

A successful attacker may be able to:

- access template context data,
- disclose application secrets,
- read, modify, or delete files available to the process,
- execute server-side Ruby code,
- potentially start operating-system processes.

The exact impact depends on runtime exposure and process permissions.

## Root cause

The application constructs ERB template source using untrusted request input. This causes user-controlled data to cross the data/code boundary and become executable template syntax.

## Remediation

- Stop constructing ERB source from request values.
- Use a static, developer-controlled template.
- Pass display values as data variables only.
- Prefer a fixed status identifier mapped to a server-controlled message.
- Minimise objects and helpers exposed to the template.
- Apply least privilege to the application process.
- Return generic template errors to clients.

## Regression tests

- Verify ERB-looking input is returned as text or rejected, never evaluated.
- Verify no filesystem or process side effects occur from message values.
- Verify valid out-of-stock rendering still works.
- Add code-review or SAST checks for dynamic template compilation involving request data.

## References

- PortSwigger SSTI overview: https://portswigger.net/web-security/server-side-template-injection
- Completed lab: https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic
