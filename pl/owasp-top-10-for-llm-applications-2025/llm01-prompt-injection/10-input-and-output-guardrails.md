# Input and Output Guardrails

## Input guardrails

Input guardrails mogą wykrywać podejrzane instrukcje, klasyfikować treść, ograniczać format albo blokować oczywiste próby nadużyć.

Pomagają, ale nie powinny być jedyną kontrolą.

## Output guardrails

Output guardrails mogą sprawdzać:

- czy model nie ujawnia danych wrażliwych,
- czy output ma oczekiwany format,
- czy nie zawiera niebezpiecznego HTML, SQL, shell commands albo tool arguments,
- czy wymaga human approval.

## Ważne rozróżnienie

Guardrail wykrywający tekst nie jest tym samym co authorization. Uprawnienia do danych i narzędzi muszą być egzekwowane w kodzie aplikacji.

## Downstream safety

Model output traktuj jak niezaufany input, szczególnie jeśli trafia do:

- HTML rendering,
- SQL,
- shell/process APIs,
- tool routers,
- email,
- workflow automation.
