# OWASP Top 10 for LLM Applications 2025

This folder contains practical security notes for applications that use Large Language Models, Retrieval-Augmented Generation (RAG), agents, tools, and private data.

The goal is not to treat the model as an isolated security product. The goal is to review the full application:

```text
user or external content
        ↓
application prompt construction
        ↓
LLM or agent
        ↓
retrieval, tools, APIs, files, or databases
        ↓
output handling and real-world actions
```

## Current coverage

- [LLM01:2025 Prompt Injection](./llm01-prompt-injection/README.md)

## Related classic OWASP mapping

Prompt Injection is an injection-style risk, but OWASP documents it separately under the OWASP Top 10 for LLM Applications. The classic OWASP Top 10:2025 A05 Injection notes contain only a short awareness mapping and point back to the full LLM01 material.

## Primary sources

- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/)
