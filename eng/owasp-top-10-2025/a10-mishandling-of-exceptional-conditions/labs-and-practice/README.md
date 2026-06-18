# A10 Labs and Practice

A10 did not have a dedicated TryHackMe lab in the room I first checked, so these practice notes document the practical review exercises completed during the sprint.

## Completed practice

1. [API error-handling review](01-api-error-handling-review.md)
2. [Transaction and rollback review](02-transaction-and-rollback-review.md)
3. [Missing and malformed parameters](03-missing-and-malformed-parameters.md)
4. [Timeout, retry and unknown outcome](04-timeout-retry-and-unknown-outcome.md)
5. [Duplicate submit and race-condition basics](05-duplicate-submit-and-race-basics.md)

## Goal

The goal was not to become a senior backend, SRE or distributed-systems engineer. The goal was to practise AppSec review questions from a Frontend Engineer perspective:

```text
What failed?
What state already changed?
Did the system fail open or fail closed?
Was the operation partially completed?
Could retry duplicate side effects?
Does the frontend show a confirmed state or only an unconfirmed state?
What backend behaviour must preserve the security invariant?
```

## Final result

PASS — I can recognise dangerous exceptional-condition handling and propose practical requirements and regression tests at Frontend Engineer -> AppSec level.
