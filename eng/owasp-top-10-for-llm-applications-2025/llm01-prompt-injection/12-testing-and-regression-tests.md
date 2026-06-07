# Testing and Regression Tests

## Test objectives

Prompt Injection testing should verify more than whether the model can be made to say a forbidden phrase. Test whether the application preserves trust boundaries, authorization, and safe tool execution when model behaviour changes.

## Direct Prompt Injection tests

- simple override,
- paraphrased override,
- policy-style markup,
- fake dialogue,
- multi-turn shaping,
- role or persona changes,
- delimiter confusion,
- input in every user-controlled field.

## Indirect Prompt Injection tests

Place harmless marker instructions in authorised test content:

- email,
- PDF or document,
- support ticket,
- webpage,
- RAG knowledge record,
- README or code comment,
- database field,
- tool result.

Confirm whether the marker changes the requested task.

## Authorization regression tests

Test with at least two users with different access:

- User A must not retrieve User B's documents.
- Shared content must remain accessible as designed.
- Prompt Injection must not broaden retrieval scope.
- Model-generated filters must not replace server-side authorization.

## Tool regression tests

For each tool, verify:

- only allowlisted tools are callable,
- all arguments match a strict schema,
- IDs and destinations are authorized,
- dangerous values are rejected,
- approval is required where expected,
- read-only roles cannot perform writes,
- action limits are enforced,
- failed calls do not leak sensitive details.

## Output-handling tests

- HTML is encoded or sanitised for the destination,
- generated URLs are validated,
- SQL is not executed as raw model text,
- shell commands are not built from model output,
- malformed JSON is rejected,
- unexpected fields are rejected,
- secrets and PII are detected where required.

## Guardrail tests

Measure both security and usability:

- known malicious patterns,
- paraphrases,
- structured content,
- indirect content,
- legitimate security/code-review input,
- multilingual business input,
- long documents,
- model and guardrail version changes.

## Monitoring tests

Verify that logs contain:

- user identity,
- source content identifiers,
- guardrail decision,
- tool name and validated arguments,
- approval result,
- execution result,
- final object or state reference.

Do not log secrets or full sensitive content unnecessarily.

## Evidence standard

A regression test should state whether it verifies:

1. model behaviour,
2. data retrieval,
3. proposed tool call,
4. executed tool call,
5. backend state change,
6. security impact.

## When to rerun

Rerun representative tests after changes to:

- model version,
- system prompt,
- prompt template,
- RAG source or chunking,
- guardrail model or rules,
- tool definitions,
- permissions,
- output parser,
- application workflow.
