# LLM01 Prompt Injection Awareness Checklist

## Architecture and trust boundaries

- [ ] What instructions are trusted?
- [ ] Where are system and developer instructions stored?
- [ ] Is user input passed separately from trusted instructions?
- [ ] Is retrieved or external content clearly labelled as untrusted?
- [ ] Which ingestion surfaces can add content to the context window?
- [ ] Can users or external parties modify those sources?

## Data access

- [ ] What private or sensitive data can the model access?
- [ ] Is retrieval scoped to the current user's authorization?
- [ ] Are public, internal, confidential, and restricted data separated?
- [ ] Can the model access more data than the user?
- [ ] Are secrets excluded from system prompts and model context unless strictly required?

## Tools and actions

- [ ] What tools or APIs can the model call?
- [ ] Are tool permissions limited by least privilege?
- [ ] Are tool names allowlisted?
- [ ] Are arguments validated against strict schemas?
- [ ] Is authorization checked for every tool call?
- [ ] Are high-risk actions approved by a human?
- [ ] Are destructive actions reversible where possible?
- [ ] Are action and rate limits enforced?

## Output handling

- [ ] Is model output treated as untrusted input?
- [ ] Is output encoded or sanitised for its destination?
- [ ] Is generated SQL prevented from becoming raw executable SQL?
- [ ] Is generated shell content prevented from reaching command execution?
- [ ] Are URLs, HTML, and structured outputs validated?

## Guardrails

- [ ] Are direct user messages checked?
- [ ] Are retrieved documents, RAG chunks, and tool outputs also checked?
- [ ] Are input and output guardrails both present where needed?
- [ ] Are false positives tested with legitimate business and security content?

## Evidence and operations

- [ ] Can logs distinguish model output from actual tool execution?
- [ ] Are retrieval and tool actions auditable?
- [ ] Are unusual output volume and repeated bypass attempts monitored?
- [ ] Do findings separate possible impact from confirmed impact?
- [ ] Are direct and indirect Prompt Injection regression tests automated where practical?
