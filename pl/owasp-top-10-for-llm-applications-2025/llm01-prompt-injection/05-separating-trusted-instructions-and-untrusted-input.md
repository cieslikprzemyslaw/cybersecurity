# Oddzielanie zaufanych instrukcji i niezaufanego inputu

## Zasada

Instrukcje systemowe i developerskie powinny być jasno oddzielone od danych użytkownika oraz retrieved content. To nie jest pełna granica bezpieczeństwa, ale zmniejsza ryzyko pomieszania ról.

## Praktyczny wzorzec

```text
Trusted instructions:
- role
- allowed task
- security policy
- output contract

Untrusted content:
- user message
- retrieved document
- webpage text
- tool result
```

## Czego unikać

- Łączenia policy i dokumentów w jednym nieopisanym bloku.
- Pozwalania retrieved content na zmianę instrukcji.
- Wstawiania sekretów do promptu, jeśli model ich nie potrzebuje.
- Ufania modelowi, że sam rozpozna, które instrukcje są zaufane.

## Dobre praktyki

- Oznaczaj zewnętrzną treść jako untrusted.
- Cytuj ją jako dane do analizy, nie jako instrukcje.
- Używaj allowlistowanych narzędzi i schema validation.
- Egzekwuj authorization poza modelem.
- Testuj direct i indirect injection jako osobne scenariusze.
