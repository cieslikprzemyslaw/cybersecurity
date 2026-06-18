# Transactions, Retries and Idempotency

## Partial state

Partial state występuje wtedy, gdy jedna część multi-step operation się uda, a kolejna zawiedzie.

```text
create evidence database record -> success
save file to storage -> success
update assessment evidence count -> failure
API returns 500
```

Frontend widzi błąd, ale backend mógł już zmienić stan.

## Logical transaction

Logical transaction to grupa zmian, które powinny succeed or fail together, np. payment + order, evidence record + file, final report status + immutable snapshot, password change + session invalidation.

## Rollback and cleanup

Database rollback może cofnąć zmiany w bazie danych w ramach transakcji. Nie cofa automatycznie plików, emaili, third-party API actions, queue messages ani external payment operations.

Na poziomie Frontend -> AppSec klucz to rozpoznać:

```text
some steps succeeded
some steps failed
system may be inconsistent
retry may duplicate side effects
```

## Retry

Retry nie jest automatycznie safe. Może zdublować payment, email, file upload, activity event albo password reset completion.

## Idempotent behaviour

Idempotent operation oznacza, że powtórzenie tego samego requestu prowadzi do tego samego final state, nie do ponownego side effect.

```text
PATCH status=completed
first request -> changes status and creates one completion event
second same request -> returns completed state without creating another event
```

## Pytania review

1. Które kroki tworzą jedną logical operation?
2. Co wydarzyło się przed errorem?
3. Co można automatycznie cofnąć?
4. Które external side effects wymagają cleanup/reconciliation?
5. Czy retry jest safe?
6. Czy duplicate requests mogą stworzyć duplicate records, emails, payments albo logs?
7. Co użytkownik powinien widzieć, gdy outcome is unknown?
