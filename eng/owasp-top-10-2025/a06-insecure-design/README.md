# A06: Insecure Design

This folder contains my learning notes for OWASP Top 10 2025 - A06: Insecure Design.

## Start Here

- [Overview](01-overview.md)
- [Labs and practice](02-labs-or-practice.md)
- [Design review checklist](03-checklist.md)
- [Regression test ideas](04-regression-tests.md)
- [Learning notes](05-learning-notes.md)

## Topic Scope

This section covers:

- security requirements and unsafe assumptions,
- design flaws versus implementation defects and misconfiguration,
- business logic and abuse cases,
- client/server trust boundaries,
- server-side enforcement,
- workflow and state-transition design,
- lightweight threat modelling,
- STRIDE as a supporting checklist,
- testable mitigations and verification.

## Completed Practice

- [Excessive trust in client-side controls](labs-and-practice/01-excessive-trust-in-client-side-controls.md)
- [Flawed enforcement of business rules](labs-and-practice/02-flawed-enforcement-of-business-rules.md)
- [Fictional Event Booking Platform threat model](labs-and-practice/03-fictional-event-platform-threat-model.md)
- [Threat register](labs-and-practice/threat-register.md)
- [Practice summary](labs-and-practice/summary.md)

The visual model is available as:

- [fictional-event-platform-complete-threat-model.drawio](labs-and-practice/fictional-event-platform-complete-threat-model.drawio)

## Direct Learning Links

- [OWASP A06:2025 - Insecure Design](https://owasp.org/Top10/2025/A06_2025-Insecure_Design/)
- [TryHackMe - Application Design Flaws](https://tryhackme.com/room/owasptopten2025two)
- [PortSwigger - Business logic vulnerabilities](https://portswigger.net/web-security/logic-flaws)
- [Lab: Excessive trust in client-side controls](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)
- [Lab: Flawed enforcement of business rules](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-flawed-enforcement-of-business-rules)

## File Roles

- `01-overview.md` is the concise source of truth for A06 concepts and distinctions.
- `02-labs-or-practice.md` indexes the completed legal practice without repeating full lab evidence.
- `03-checklist.md` is a practical design-review checklist.
- `04-regression-tests.md` contains reusable fix-verification ideas.
- `05-learning-notes.md` records corrections and personal consolidation.
- `labs-and-practice/` contains the detailed lab evidence and the fictional threat model.
- `security-findings/` contains a developer-facing example finding.

## Example Finding

- [Client-controlled product price accepted by checkout](security-findings/01-client-controlled-product-price.md)
