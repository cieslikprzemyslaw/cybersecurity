# Prompt Injection Overview

## Root cause

Prompt Injection exploits ambiguity between instructions and data. The model may interpret attacker-controlled content as a command even when the application intended it to be only text to translate, summarise, classify, or analyse.

## Direct and indirect injection

### Direct Prompt Injection

The attacker supplies instructions directly through a prompt or another user-controlled field.

```text
attacker message → application → model behaviour changes
```

### Indirect Prompt Injection

The attacker plants instructions in external content that the model later retrieves or processes.

```text
attacker-controlled document/email/page
→ innocent user request
→ application retrieves the content
→ model processes hidden instruction
→ behaviour changes
```

## Why it matters

A model that only answers public weather questions has a relatively small blast radius. A model connected to private documents, email, calendars, code execution, internal APIs, or production systems can cause substantially greater harm.

Possible consequences include:

- manipulated or false output,
- disclosure of sensitive information,
- system prompt or internal logic disclosure,
- unauthorised data retrieval,
- unsafe tool calls,
- message or ticket creation,
- file modification,
- command execution,
- influence over business decisions.

## Prompt Injection is not the same as impact

A successful behavioural change confirms Prompt Injection. It does not automatically prove a data breach or system compromise.

```text
model says it performed an action
≠
application actually performed the action
```

Verify the tool logs, API response, database state, created object, sent message, or other downstream evidence.

## Why stronger prompts are not a complete fix

System prompts are made of the same kind of natural-language tokens as user and retrieved content. Tight instructions can raise the attack cost, but security must not depend only on the model choosing the correct instruction.

## Primary source

- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
