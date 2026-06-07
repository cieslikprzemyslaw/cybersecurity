# Learning Notes

## Current learning path

The topic was studied as practical Prompt Injection awareness for a Frontend Engineer transitioning into AppSec, not as a full offensive AI-security course.

Resources used:

- TryHackMe Prompt Injection theory and practical agent,
- TryHackMe Prompt Defence,
- OWASP LLM01:2025 Prompt Injection,
- developer-focused prompt structuring guidance.

## Mental model developed

```text
untrusted content
→ model context
→ changed instruction-following behaviour
→ possible data access or action
```

The model receives system instructions, developer instructions, user input, retrieved content, and tool results inside a context window. Role labels and formatting help separate these sources, but they are not an unbreakable security boundary.

## Practical observation: LLMborghini

Baseline:

- the chatbot priced the car at USD 750,000,
- a normal request for a USD 1 price was rejected.

A policy-style XML block caused the model to agree to the USD 1 price.

Initial practical lesson:

- angle brackets are not magic,
- structured content can appear authoritative to the model,
- attacker-controlled markup may change behaviour,
- a changed response proves Prompt Injection behaviour,
- it does not prove that a backend sale occurred.

## Direct techniques covered

- simple overrides,
- paraphrased overrides,
- format-based injection,
- simulated dialogue,
- multi-turn prompt shaping.

## Indirect Prompt Injection understanding

Malicious instructions may be hidden in:

- webpages,
- documents,
- emails,
- support tickets,
- tool output,
- RAG content,
- code repositories.

An innocent user request may activate the injection. The attacker may only need to plant the content.

## Defence understanding

Prompt Injection is probabilistic and cannot be completely solved by a stronger system prompt. Defence requires multiple layers:

1. system prompt hardening,
2. separate trusted instructions and untrusted input,
3. input guardrails,
4. checks on retrieved and external content,
5. output guardrails and schema validation,
6. least-privilege data and tool access,
7. current-user authorization,
8. approval for dangerous actions,
9. logging, monitoring, and limits.

## Important evidence distinction

```text
model output
≠
tool execution
≠
backend state change
≠
confirmed security impact
```

## Remaining checkpoint topics

Before marking the topic complete, confirm the ability to explain:

- Prompt Injection in own words,
- direct versus indirect injection,
- why tools and private data increase risk,
- why system prompts and delimiters are not complete fixes,
- how user input and retrieved content should be separated,
- what least privilege means for agents and RAG,
- what evidence proves actual impact,
- which regression tests should exist.
