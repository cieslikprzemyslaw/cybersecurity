# Indirect Prompt Injection

## Definition

Indirect Prompt Injection occurs when malicious instructions are hidden in external content that an LLM or agent later ingests. The attacker does not need to type the instruction into the victim's chat session.

## Common ingestion surfaces

- webpages and hidden HTML,
- comments or invisible text,
- emails,
- PDFs and office documents,
- support tickets,
- database records,
- code repositories and README files,
- configuration files,
- tool outputs,
- shared documents,
- RAG knowledge-base chunks,
- multimodal content such as images containing text.

Every automatic ingestion surface creates a trust boundary.

## Typical flow

```text
1. Attacker plants malicious content.
2. The application stores or retrieves it.
3. An innocent user asks for a summary or analysis.
4. The content enters the context window.
5. The model treats part of the content as an instruction.
6. Output, data access, or tool behaviour changes.
```

## Zero-click characteristic

In a zero-click scenario, the attacker only needs to plant the malicious content. Normal application processing or an innocent user request activates it without further attacker interaction.

## Why it is dangerous

A guardrail that checks only the direct user message may run before external content is retrieved. The malicious instruction then enters from a different path and avoids that initial check.

Indirect Prompt Injection can lead to:

- manipulated summaries or recommendations,
- phishing or scam content,
- unauthorised data retrieval,
- private-data disclosure,
- unsafe tool calls,
- file or code modification,
- command execution when an over-privileged agent is connected.

## AppSec review questions

1. Which external sources can enter the model context?
2. Who can modify each source?
3. Is the source labelled as untrusted data?
4. Does it pass through an appropriate guardrail?
5. Is retrieval scoped to the current user's authorization?
6. Can the model call tools after processing the content?
7. Are tool arguments independently validated?
8. Is human approval required for high-risk actions?
9. Are retrieved content and resulting actions logged?

## Key distinction

Indirect Prompt Injection is the manipulation technique. Data leakage, code execution, or ticket creation are possible impacts that require separate evidence.
