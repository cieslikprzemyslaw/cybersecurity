# Agent, Tool, and RAG Risks

## Why connected capabilities increase risk

Prompt Injection changes model behaviour. Real harm becomes more likely when the model can access sensitive data or trigger actions through tools.

Examples of connected capabilities include:

- internal document search,
- email read/send,
- calendar access,
- support-ticket creation,
- database queries,
- file operations,
- source-code modification,
- command execution,
- internal and third-party APIs,
- RAG retrieval across private corpora.

## Review the complete capability chain

```text
untrusted content
→ model interpretation
→ selected tool
→ generated arguments
→ authorization check
→ validation
→ execution
→ resulting state change
```

A secure design must not skip the deterministic checks between model output and execution.

## Excessive Agency

Excessive Agency means the application grants more functionality, permission, or autonomy than the task requires.

Examples:

- an email summariser can also send email,
- a support assistant can delete tickets,
- a read-only coding assistant can execute shell commands,
- a RAG assistant can search documents the current user cannot access,
- a calendar assistant can invite external attendees without approval.

## RAG risks

Retrieved content is not automatically trustworthy. A knowledge base can contain malicious instructions because of:

- compromised source documents,
- attacker-controlled support tickets,
- poisoned or modified records,
- unreviewed uploaded files,
- public webpages included in retrieval,
- stale access-control metadata.

RAG improves relevance; it does not remove Prompt Injection risk.

## Secure design requirements

- scope retrieval to the current user's permissions,
- separate public, internal, confidential, and restricted data,
- label retrieved content as untrusted,
- validate source provenance and access metadata,
- use read-only scopes where possible,
- expose only task-specific tools,
- prefer narrow functions over general shell or SQL tools,
- validate tool names and arguments against allowlists and schemas,
- require human approval for irreversible or high-risk actions,
- make actions reversible where possible,
- log retrieval, tool selection, arguments, approval, and results.

## Principle

Every permission not granted is a capability an attacker cannot exploit, even when the Prompt Injection succeeds.
