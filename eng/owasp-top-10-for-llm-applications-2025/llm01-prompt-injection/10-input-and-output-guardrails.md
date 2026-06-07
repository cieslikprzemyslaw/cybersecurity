# Input and Output Guardrails

## Role in defence in depth

Guardrails add checks before and after model execution. They supplement system prompt hardening; they do not replace it.

## Blocklists and regex

A simple guardrail rejects known strings or patterns.

Advantages:

- very fast,
- inexpensive,
- useful against low-effort known attacks.

Limitations:

- paraphrasing changes the words but keeps the intent,
- encoding and obfuscation may bypass matching,
- Unicode homoglyphs and zero-width characters can evade filters,
- multilingual input may avoid expected phrases,
- blocklists do not understand context or semantic intent.

Use keyword filtering as a cheap first layer, not the primary defence.

## AI-powered classifiers

Semantic classifiers attempt to identify attack intent instead of exact strings. Meta's Llama Prompt Guard 2 is an example of a BERT-based classifier designed for prompt-injection and jailbreak detection.

Classifiers provide broader coverage than string matching, but they remain probabilistic and can be attacked with adversarial or noisy input.

## Input guardrails

Input guardrails run before the main model receives content. They may:

- reject likely Prompt Injection,
- remove or mask PII,
- enforce topic boundaries,
- reject malformed or oversized input,
- classify risk before expensive processing.

## Output guardrails

Output guardrails run after generation. They may:

- detect secrets or PII,
- detect policy violations,
- enforce output schemas,
- reject unexpected tool names,
- validate tool arguments,
- block unsafe or malformed downstream content.

## Indirect injection gap

Checking only the direct user message is insufficient. Malicious instructions may enter later through:

- retrieved documents,
- RAG chunks,
- webpages,
- emails,
- uploaded files,
- database records,
- tool outputs.

Every external content path should be treated as untrusted and checked at the appropriate point in the pipeline.

## Cascade architecture

A practical sequence is:

```text
cheap syntax and size checks
→ known-pattern checks
→ semantic classifier for higher-risk content
→ main model
→ schema and policy validation
→ tool authorization and execution
```

## Trade-offs

Guardrails must balance:

- latency,
- coverage,
- cost,
- false positives,
- false negatives,
- user experience.

Security and code-review prompts are common false-positive cases because they may contain exploit-like language for legitimate reasons.

## Testing requirements

Measure:

- bypass rate across paraphrases and formats,
- false-positive rate on legitimate business input,
- behaviour with indirect/RAG content,
- output schema enforcement,
- consistency across model or guardrail updates.
