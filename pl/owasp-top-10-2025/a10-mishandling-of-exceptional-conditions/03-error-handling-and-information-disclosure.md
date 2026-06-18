# Error Handling and Information Disclosure

## Odpowiedź zewnętrzna

Bezpieczna zewnętrzna odpowiedź błędu powinna być użyteczna dla klienta, ale nie powinna ujawniać wewnętrznej implementacji.

Może zawierać HTTP status, generyczny komunikat, stabilny application error code i request/correlation ID.

Nie powinien zawierać stack trace, SQL query, filesystem path, secrets, tokens, internal hostnames, sensitive config, raw exception object albo pełnych request/response bodies.

## Generyczna odpowiedź nie dowodzi recovery

```text
500 Something went wrong
```

może być bezpieczne pod kątem information disclosure, ale nie dowodzi rollbacku, cleanupu, braku płatności, braku wysłanego emaila, braku zmiany roli, zwolnienia zasobów ani bezpiecznego retry.

Odpowiedź pokazuje tylko to, co zobaczył klient. To nie jest dowód stanu backendu.

## Internal logs

Internal logs powinny zawierać bezpieczny kontekst: request/correlation ID, operation name, actor ID, target resource ID, safe reason code, result, timestamp i environment. Nadal nie wolno logować secrets ani zbędnych danych osobowych.

## Global error handler

Global error handler jest przydatny jako fallback i może zapobiec stack trace disclosure. Nie zastępuje lokalnej obsługi w warstwie, która rozumie operację. Nie robi automatycznie rollbacku multi-step workflow, nie usuwa częściowo zapisanych plików i nie odwraca third-party side effects.

## Try/catch nie wystarcza

Niebezpieczne wzorce:

```text
catch error -> log -> continue as success
catch error -> ignore -> return 200
catch error -> retry unsafe action repeatedly
catch error -> leave partial state
```

Bezpieczniej:

```text
catch error
    -> stop sensitive operation
    -> rollback or clean up where possible
    -> avoid continuing from unknown state
    -> return controlled response
    -> log safe internal evidence
```
