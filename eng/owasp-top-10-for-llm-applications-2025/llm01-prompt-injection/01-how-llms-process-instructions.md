# How LLMs Process Instructions

## Context window

An LLM generates a response using the information available inside its context window. Depending on the application, that context may contain:

- system instructions,
- developer instructions,
- user messages,
- previous assistant messages,
- retrieved documents or RAG chunks,
- tool outputs,
- agent observations,
- uploaded files or extracted text.

These sources are intended to remain logically separate, but they are ultimately processed as one sequence of tokens.

## Instruction hierarchy

Providers use role labels, special tokens, metadata, or message objects to communicate priority. A common intended hierarchy is:

```text
system > developer > user > assistant > tool
```

This helps the model interpret the conversation, but it is not equivalent to an operating-system permission boundary. The model has learned to follow the hierarchy; it does not enforce it as deterministic authorization logic.

## Chat templates

Examples include role-based message formats such as ChatML and provider-specific message APIs. Special markers may identify where a system, user, assistant, or tool message begins and ends.

These formats improve separation, but malicious input can imitate trusted-looking markup or dialogue. Therefore:

```text
better structure ≠ guaranteed isolation
```

## Next-token prediction

An LLM predicts the next token based on the preceding context. It does not understand authority in the same deterministic way as an access-control system. Conflicting instructions create ambiguity, and attacker-controlled text may shift the probability of an unsafe response.

This explains why Prompt Injection is different from a classic binary bug:

- there may be no single condition to patch,
- the same prompt may not always produce the same result,
- paraphrasing can change the outcome,
- model or configuration changes may alter results,
- a refusal is a learned behaviour, not a hard security rule.

## AppSec conclusion

Prompt and role separation are useful controls, but authorization, data access, tool permissions, validation, and approval must be enforced by deterministic application code.
