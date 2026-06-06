# OS Command Injection

OS Command Injection occurs when untrusted input reaches an operating-system command, shell, command interpreter, or unsafe process-execution path and changes what the system executes.

```text
user-controlled input
-> backend code
-> command construction or process arguments
-> shell or process execution
-> changed operating-system behaviour
```

## Completed learning

- TryHackMe OS Command Injection theory
- PHP `exec()` data-flow example
- Python Flask `subprocess.Popen(..., shell=True)` example
- Direct/verbose and blind command injection
- Timing, file redirection, and OAST concepts
- PortSwigger simple command-injection lab
- PortSwigger blind output-redirection lab

## Start here

1. [Overview](01-overview.md)
2. [Command execution and shells](02-command-execution-and-shells.md)
3. [Direct and blind command injection](03-direct-and-blind-command-injection.md)
4. [Remediation](04-remediation.md)
5. [Cheat sheet](cheatsheet.md)
6. [Regression tests](regression-tests.md)

## Labs

- [Simple OS command injection](labs/01-simple-command-injection.md)
- [Blind OS command injection with output redirection](labs/02-blind-command-injection-output-redirection.md)
- [Lab comparison](labs/summary.md)

## Core questions

For every suspected issue, identify:

- What feature is being tested?
- What input is controlled?
- Where does it go?
- What process API or shell may receive it?
- Is a complete command string being built?
- Is the shell involved, or are arguments passed separately?
- What facts are known and what remains an assumption?
- What evidence proves execution?
- Is the output direct, blind, time-based, file-based, or out-of-band?
- What is the root cause?
- How should the feature be implemented safely?
- What regression tests should prevent recurrence?

## Key distinction

OS Command Injection is not simply "bad validation".

The central failure is allowing untrusted data to become executable command syntax or dangerous process arguments.

The preferred fix is to remove shell execution, not to keep sending increasingly sanitised strings to a shell.

## Scope and safety

All payload examples and lab observations are for TryHackMe, PortSwigger Web Security Academy, local labs, or explicitly authorised testing only.

The notes use benign proof commands and do not treat reverse shells or destructive commands as routine detection techniques.

## References

- TryHackMe Command Injection: https://tryhackme.com/room/oscommandinjection
- PortSwigger OS Command Injection: https://portswigger.net/web-security/os-command-injection
- OWASP OS Command Injection Defense Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html
- PayloadBox list, used only as a secondary lab reference: https://github.com/payload-box/command-injection-payload-list
