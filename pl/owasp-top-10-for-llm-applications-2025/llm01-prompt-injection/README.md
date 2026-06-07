# LLM01:2025 - Prompt Injection

## Definicja

Prompt Injection występuje wtedy, gdy niezaufany input zmienia zachowanie albo output aplikacji LLM w niezamierzony sposób.

Praktyczny model:

```text
niezaufana treść
→ LLM albo agent
→ zmienione instruction-following behaviour
→ możliwy dostęp do danych, tool call albo downstream action
```

Niezaufana treść może pochodzić bezpośrednio od użytkownika albo pośrednio z web page, emaila, dokumentu, support ticketu, rekordu bazy, wyniku narzędzia, repozytorium kodu albo chunku pobranego przez RAG.

## Główna lekcja AppSec

Model nie jest granicą bezpieczeństwa. Przetwarza instrukcje i dane probabilistycznie i może pomylić niezaufaną treść z zaufanymi instrukcjami. Lepsze prompt wording może podnieść koszt ataku, ale authorization, least privilege, validation, output handling i approval gates muszą być egzekwowane poza modelem.

## Ważne rozróżnienia

| Pojęcie | Znaczenie |
|---|---|
| Prompt Injection | Technika manipulacji zmieniająca zachowanie modelu. |
| Data leakage | Jeden z możliwych skutków Prompt Injection. |
| Excessive Agency | System daje modelowi za dużo możliwości, uprawnień albo autonomii. |
| Insecure tool use | Model-generated tool arguments są zaufane albo wykonywane bez wystarczającej walidacji. |
| Model output | Wygenerowany tekst albo proponowany tool call. Sam nie dowodzi wykonania. |
| Confirmed impact | Zweryfikowany dostęp do danych, wykonanie narzędzia, zmiana stanu albo inny skutek bezpieczeństwa. |

## Notatki w tym folderze

1. [Jak LLM przetwarza instrukcje](01-how-llms-process-instructions.md)
2. [Prompt Injection overview](02-prompt-injection-overview.md)
3. [Direct Prompt Injection](03-direct-prompt-injection.md)
4. [Indirect Prompt Injection](04-indirect-prompt-injection.md)
5. [Oddzielanie zaufanych instrukcji i niezaufanego inputu](05-separating-trusted-instructions-and-untrusted-input.md)
6. [Ryzyka agentów, narzędzi i RAG](06-agent-tool-and-rag-risks.md)
7. [Dowody, impact i severity](07-evidence-impact-and-severity.md)
8. [Prompt Injection testing cheatsheet](08-prompt-injection-cheatsheet.md)
9. [System prompt hardening](09-system-prompt-hardening.md)
10. [Input i output guardrails](10-input-and-output-guardrails.md)
11. [Deployment controls i least privilege](11-deployment-controls-and-least-privilege.md)
12. [Testing i regression tests](12-testing-and-regression-tests.md)
13. [Awareness checklist](awareness-checklist.md)
14. [Learning notes](learning-notes.md)

## Główne źródło

- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
