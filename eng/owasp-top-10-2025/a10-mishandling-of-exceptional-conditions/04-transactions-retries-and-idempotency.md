# Transactions, Retries and Idempotency

## Partial state

Partial state occurs when one part of a multi-step operation succeeds and another part fails.

Example:

```text
create evidence database record -> success
save file to storage -> success
update assessment evidence count -> failure
API returns 500
```

The frontend sees failure, but the backend may already have changed state.

## Logical transaction

A logical transaction is the group of changes that must succeed or fail together for the application to remain valid.

Examples:

- payment recorded and order created,
- evidence record created and evidence file stored,
- report marked final and immutable snapshot saved,
- password changed and active sessions invalidated,
- user deleted and related security records handled.

## Rollback and cleanup

Database rollback can undo database changes inside the transaction. It does not automatically undo:

- files already written,
- emails already sent,
- third-party API actions,
- queue messages,
- external payment operations.

At Frontend -> AppSec level, the key skill is recognising the risk:

```text
some steps succeeded
some steps failed
system may be inconsistent
retry may duplicate side effects
```

## Retry

Retry is not automatically safe.

Potential duplicate side effects:

- payment created twice,
- email sent twice,
- file uploaded twice,
- activity event created twice,
- password reset token used twice.

## Idempotent behaviour

An operation is idempotent when repeating the same request leads to the same intended final state rather than repeating the side effect.

Example:

```text
PATCH status=completed
first request -> changes status and creates one completion event
second same request -> returns completed state without creating another event
```

## Review questions

1. Which steps form one logical operation?
2. Which steps happened before the error?
3. Which changes can be rolled back automatically?
4. Which external side effects need cleanup or reconciliation?
5. Is retry safe?
6. Could duplicate requests create duplicate records, emails, payments or logs?
7. What should the user see while the outcome is unknown?
