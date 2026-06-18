# Fail-open and Fail-closed

## Fail-open

Fail-open oznacza, że aplikacja pozwala na sensitive operation, gdy security control nie może podjąć bezpiecznej decyzji.

```text
authorization service timeout
    -> backend cannot verify permission
    -> backend allows the request anyway
    -> protected state is changed
```

Aplikacja traktuje niepewność jak permission.

## Fail-closed

Fail-closed oznacza, że aplikacja odrzuca albo zatrzymuje sensitive operation, gdy nie może potwierdzić security rule.

```text
authorization service timeout
    -> backend cannot verify permission
    -> backend denies the operation
    -> protected state is not changed
    -> internal evidence is logged
```

Fail-closed nie oznacza crasha całej aplikacji. Oznacza zachowanie reguły bezpieczeństwa i powrót do znanego bezpiecznego stanu.

## Pytania review

1. Jaka security rule musi pozostać prawdziwa?
2. Która dependency albo check może failować?
3. Co się dzieje, gdy aplikacja nie może potwierdzić tej reguły?
4. Czy kod mimo tego pozwala na akcję?
5. Czy system wraca do znanego bezpiecznego stanu?

## Typowe fail-open patterns

- authorization service fails and access is allowed,
- session lookup fails and user is treated as authenticated,
- file scanner fails and upload is accepted as safe,
- permission lookup fails and admin behaviour is enabled,
- catch block logs error but continues operation,
- validation throws and unsafe defaults are used.

## Frontend note

UI nie może pokazywać wrażliwej operacji jako sukcesu bez potwierdzenia z backendu. Po timeout bezpieczny stan UI to `unable to confirm`, a następnie odświeżenie autorytatywnego stanu backendu.
