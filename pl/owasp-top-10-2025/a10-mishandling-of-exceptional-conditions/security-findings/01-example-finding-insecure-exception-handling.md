# Example Finding: Insecure Exceptional-condition Handling Can Leave Partial Evidence State

## Summary

Workflow uploadu evidence może zostawić partial state, jeżeli jeden krok się uda, a późniejszy zawiedzie. Generyczna odpowiedź `500` wraca do klienta, ale nie dowodzi, że backend wykonał rollback albo cleanup.

## Affected workflow

```text
POST /api/assessments/{assessmentId}/evidence-file
```

Normal flow:

```text
create evidence record
save evidence file
update assessment evidence count
return 201 Created
```

## Vulnerable behaviour

Jeżeli aktualizacja podsumowania assessmentu zawiedzie po utworzeniu evidence record i zapisaniu pliku, API może zwrócić generyczne `500`. Taka odpowiedź ogranicza information disclosure, ale nie dowodzi, że rekord albo plik zostały usunięte.

## Security impact

Aplikacja może zachować inconsistent state:

- evidence record exists but assessment summary does not reflect it,
- file exists without complete controlled workflow,
- UI shows failure while backend has partial data,
- retry may create duplicates,
- reports or audit review may use inconsistent data.

Affected properties: integrity, accountability and availability.

## Evidence to collect

- request wywołujący awarię w kontrolowanym teście,
- status i body odpowiedzi,
- stan bazy przed i po,
- stan file storage przed i po,
- assessment summary przed i po,
- logi z request/correlation ID.

Nie zakładaj korupcji danych bez potwierdzenia, że stan faktycznie się zmienił.

## Recommendation

- Treat upload as one logical operation.
- Roll back database changes where possible.
- Clean up or quarantine files when later steps fail.
- Return controlled errors without internal details.
- Log safe context and correlation ID.
- Ensure retry does not create duplicate active records.
- Add regression tests for each failing step.

## Regression tests

- Simulate failure after evidence record creation and verify no active partial evidence remains.
- Simulate failure after file save and verify file is removed, quarantined or marked for cleanup.
- Verify controlled `500` without stack trace, SQL query, path or secrets.
- Verify retry does not create duplicate active records.

## Interview-style takeaway

> Generic error response jest przydatny, ale nie jest dowodem bezpiecznego odzyskania stanu. Podczas review sprawdziłbym, jaki stan zmienił się przed awarią, czy wystąpił rollback albo cleanup, czy retry może zdublować dane i jaki test dowodzi powrotu do known safe state.
