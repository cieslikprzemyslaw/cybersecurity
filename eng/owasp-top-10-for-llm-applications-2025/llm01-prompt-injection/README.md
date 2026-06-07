# LLM01:2025 — Prompt Injection

## Definition

Prompt Injection occurs when untrusted input changes an LLM application's behaviour or output in an unintended way.

A useful mental model is:

```text
untrusted content
→ LLM or agent
→ changed instruction-following behaviour
→ possible data access, tool call, or downstream action
```

The untrusted content may be supplied directly by a user or indirectly through a webpage, email, document, support ticket, database record, tool result, code repository, or RAG-retrieved chunk.

## Core AppSec lesson

The model is not a security boundary. It processes instructions and data probabilistically, and it may confuse untrusted content with trusted instructions. Prompt wording can raise the attack cost, but authorization, least privilege, validation, output handling, and approval gates must be enforced outside the model.

## Important distinctions

| Concept | Meaning |
|---|---|
| Prompt Injection | The manipulation technique that changes model behaviour. |
| Data leakage | One possible impact of Prompt Injection. |
| Excessive Agency | The system gives the model too much capability, permission, or autonomy. |
| Insecure tool use | Model-generated tool arguments are trusted or executed without sufficient validation. |
| Model output | Generated text or a proposed tool call. It does not prove execution. |
| Confirmed impact | A verified data access, tool execution, state change, or other security consequence. |

## Notes in this folder

1. [How LLMs process instructions](./01-how-llms-process-instructions.md)
2. [Prompt Injection overview](./02-prompt-injection-overview.md)
3. [Direct Prompt Injection](./03-direct-prompt-injection.md)
4. [Indirect Prompt Injection](./04-indirect-prompt-injection.md)
5. [Separating trusted instructions and untrusted input](./05-separating-trusted-instructions-and-untrusted-input.md)
6. [Agent, tool, and RAG risks](./06-agent-tool-and-rag-risks.md)
7. [Evidence, impact, and severity](./07-evidence-impact-and-severity.md)
8. [Prompt Injection testing cheatsheet](./08-prompt-injection-cheatsheet.md)
9. [System prompt hardening](./09-system-prompt-hardening.md)
10. [Input and output guardrails](./10-input-and-output-guardrails.md)
11. [Deployment controls and least privilege](./11-deployment-controls-and-least-privilege.md)
12. [Testing and regression tests](./12-testing-and-regression-tests.md)
13. [Awareness checklist](./awareness-checklist.md)
14. [Learning notes](./learning-notes.md)

## Primary source

- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
