# A10: Mishandling of Exceptional Conditions

This folder contains my practical notes for OWASP Top 10 2025 — A10: Mishandling of Exceptional Conditions.

## Status

PASS — Frontend Developer transitioning into AppSec

Completed work:

- OWASP A10:2025 theory review,
- OWASP Error Handling Cheat Sheet review,
- API error-handling review,
- partial-state and rollback review,
- missing and malformed parameter review,
- timeout, retry and unknown-outcome review,
- duplicate submit / race-condition basics,
- comparison of A10 with A01 Broken Access Control, A06 Insecure Design and A09 Security Logging and Alerting Failures.

This status does not mean expert-level distributed-systems, database-transaction, SRE or concurrency knowledge. It means I can recognise dangerous failure behaviour, distinguish fail-open from fail-closed, review basic error handling, identify unsafe partial states, suggest practical security requirements and propose useful regression tests.

## Core mental model

```text
unexpected condition
    -> detection
    -> safe handling
    -> rollback / cleanup / recovery
    -> generic external response
    -> internal logging and alerting
```

## Main review questions

1. What was supposed to happen?
2. What abnormal condition occurred?
3. What state had already changed?
4. Did the system fail open or fail closed?
5. Was the operation partially completed?
6. Was rollback, cleanup or recovery required?
7. Did the response expose internal details?
8. Could retry duplicate a side effect?
9. Could the same condition exhaust resources or be abused repeatedly?
10. What regression test proves the system returns to a safe state?

## Frontend perspective

Frontend controls improve UX, but they are not authoritative security controls.

- React error boundaries improve UI resilience, not backend recovery.
- A generic frontend error does not prove rollback.
- A `500` response does not prove that nothing changed.
- A timeout means the frontend did not receive confirmation; the server may still have processed the request.
- Optimistic UI must not leave a sensitive action displayed as successful when the backend rejects or cannot confirm it.
- Disabled buttons reduce accidental double-clicks, but they do not stop Burp, ZAP, Postman, DevTools, scripts, retries or two browser tabs.
- Backend validation, authentication, authorization, state transitions and transaction behaviour remain the source of truth.

## Start here

- [Overview](01-overview.md)
- [Fail-open and fail-closed](02-fail-open-and-fail-closed.md)
- [Error handling and information disclosure](03-error-handling-and-information-disclosure.md)
- [Transactions, retries and idempotency](04-transactions-retries-and-idempotency.md)
- [Timeouts, resources and race conditions](05-timeouts-resources-and-race-conditions.md)
- [Review checklist](06-checklist.md)
- [Regression tests](07-regression-tests.md)
- [Learning notes](08-learning-notes.md)
- [Labs and practice](labs-and-practice/README.md)
- [Example security finding](security-findings/01-example-finding-insecure-exception-handling.md)


## Practice evidence

The practical part of this topic is documented in separate files:

- [API error-handling review](labs-and-practice/01-api-error-handling-review.md)
- [Transaction and rollback review](labs-and-practice/02-transaction-and-rollback-review.md)
- [Missing and malformed parameters](labs-and-practice/03-missing-and-malformed-parameters.md)
- [Timeout, retry and unknown outcome](labs-and-practice/04-timeout-retry-and-unknown-outcome.md)
- [Duplicate submit and race-condition basics](labs-and-practice/05-duplicate-submit-and-race-basics.md)

## Direct learning links

- OWASP A10:2025 — Mishandling of Exceptional Conditions: https://owasp.org/Top10/2025/A10_2025-Mishandling_of_Exceptional_Conditions/
- OWASP Error Handling Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html
