# System Prompt Hardening

## Rola system promptu

System prompt może wyjaśniać rolę modelu, zakres zadania, zasady bezpieczeństwa i format outputu. Jest ważny, ale nie jest samodzielną kontrolą bezpieczeństwa.

## Dobre praktyki

- Wyraźnie rozdziel trusted instructions od untrusted content.
- Powiedz modelowi, że retrieved content może zawierać złośliwe instrukcje.
- Ustal output contract.
- Ogranicz model do opisywania działań, jeśli nie ma prawa ich wykonać.
- Wymagaj cytowania źródeł tam, gdzie ma to sens.

## Ograniczenia

Prompt hardening nie zastępuje:

- authorization,
- access control,
- tool permission checks,
- schema validation,
- approval gates,
- logging i monitoring.

## Najważniejsza zasada

Jeśli złamanie system promptu daje realny dostęp do danych albo narzędzi, to problemem jest architektura aplikacji, nie tylko prompt wording.
