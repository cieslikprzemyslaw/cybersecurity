# Deployment Controls and Least Privilege

## Assume a prompt-layer control will fail

Some injections will bypass prompt hardening and guardrails. Deployment controls determine what a compromised model can actually access or do.

## Principle of Least Privilege

Every model, agent, tool, and retrieval component should receive only the permissions required for its task.

Examples:

- a summariser receives read-only access,
- a draft-writing assistant cannot send messages,
- a ticket assistant can create but not delete tickets,
- a coding assistant cannot access production credentials,
- a RAG system retrieves only documents the current user may view.

## Enforce current-user authorization

The model must not gain broader data access than the requesting user.

```text
user identity
→ authorization check
→ scoped retrieval
→ model context
```

Do not retrieve the full corpus and ask the model to hide unauthorized results. Authorization must happen before content reaches the model.

## Tool design

Prefer narrow, task-specific tools:

```text
create_support_ticket(title, category, body)
```

over broad capabilities:

```text
execute_shell(command)
```

Controls should include:

- allowlisted tool names,
- strict JSON or typed schemas,
- server-side validation,
- authorization for every call,
- constrained values and destinations,
- action and rate limits,
- timeouts,
- human approval for high-risk operations,
- reversible operations where possible.

## Treat model output as untrusted

OWASP LLM05:2025 Improper Output Handling concerns insufficient validation, sanitisation, and handling of model output before it reaches downstream systems.

Potential chains include:

```text
LLM-generated JavaScript → unsafe browser rendering → XSS
LLM-generated SQL → non-parameterised database execution → SQL Injection
LLM-generated shell value → exec() or shell → command execution / RCE
```

Before downstream use:

- validate structure,
- enforce schemas,
- encode for the destination context,
- sanitise HTML when HTML is explicitly required,
- use parameterised queries,
- avoid shell execution,
- validate tool arguments independently of the model.

## Logging and monitoring

Record enough information to reconstruct security events:

- requesting user and authorization context,
- retrieved sources and document identifiers,
- selected tool,
- validated arguments,
- approval result,
- execution result,
- created or changed object identifiers,
- guardrail decisions,
- unusual output volume or repeated bypass attempts.

## Rate and action limits

Limit:

- requests per user,
- tokens per request or session,
- tool calls per task,
- records retrieved,
- recipients or destinations,
- repeated failed approval or validation attempts.

These controls reduce blast radius and improve detection; they do not replace prevention.
