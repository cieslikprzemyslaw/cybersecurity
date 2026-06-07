# OWASP Top 10 for LLM Applications 2025

Ten katalog zawiera praktyczne notatki bezpieczeństwa dla aplikacji używających Large Language Models, RAG, agentów, narzędzi i prywatnych danych.

Celem nie jest traktowanie modelu jako osobnego produktu bezpieczeństwa. Celem jest review całej aplikacji:

```text
użytkownik albo zewnętrzna treść
        ↓
budowanie promptu przez aplikację
        ↓
LLM albo agent
        ↓
retrieval, tools, API, pliki albo bazy danych
        ↓
obsługa outputu i realne działania
```

## Aktualne pokrycie

- [LLM01:2025 Prompt Injection](llm01-prompt-injection/README.md)

## Powiązanie z klasycznym OWASP

Prompt Injection jest ryzykiem w stylu injection, ale OWASP dokumentuje je osobno w OWASP Top 10 for LLM Applications. Klasyczny dział A05 Injection zawiera tylko krótkie mapowanie awareness i odsyła tutaj po pełne notatki LLM01.

## Główne źródła

- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/)
