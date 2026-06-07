# AI Prompt Injection Awareness

## Mapping decision

Prompt Injection is an injection-style security risk, but it is not documented here as a full classic web-injection topic.

OWASP Top 10:2025 A05 Injection explicitly notes that a related class of injection vulnerabilities exists in LLM applications and points to **OWASP LLM01:2025 Prompt Injection** for the dedicated guidance.

Full notes:

```text
eng/owasp-top-10-for-llm-applications-2025/
  llm01-prompt-injection/
```

## Why it is related to A05

The shared high-level idea is that untrusted input changes how an interpreter processes a task.

```text
Classic injection:
untrusted input → browser/database/shell/template interpreter → unintended execution

Prompt Injection:
untrusted content → LLM or agent → unintended instruction-following behaviour
```

## Important difference

SQL, OS command, and many template injections cross a deterministic grammar or code/data boundary. Prompt Injection operates through probabilistic interpretation of natural-language context. The security impact also depends heavily on the surrounding application, connected data, tools, and permissions.

## AppSec awareness points

- treat the model as untrusted and non-deterministic,
- separate trusted instructions from user and retrieved content,
- treat all external content as untrusted data,
- never use a prompt as authorization,
- enforce access control before retrieval,
- restrict tools with least privilege,
- validate tool arguments before execution,
- treat model output as untrusted input,
- require approval for high-risk actions,
- distinguish generated output from actual execution and impact.

## References

- [OWASP A05:2025 Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
- [OWASP LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
