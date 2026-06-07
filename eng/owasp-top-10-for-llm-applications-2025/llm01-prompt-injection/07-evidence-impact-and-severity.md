# Evidence, Impact, and Severity

## Why evidence must be separated from assumptions

LLM responses can sound confident and can claim that an action occurred. AppSec findings must distinguish generated text from verified application behaviour.

## Evidence ladder

```text
1. Attacker-controlled content reached the application.
2. The content entered the model context.
3. Model behaviour changed.
4. The model proposed or generated a tool call.
5. The application accepted the tool call.
6. The tool executed successfully.
7. Data was accessed or application state changed.
8. A real security or business impact occurred.
```

Each step requires its own evidence.

## Examples

| Observation | What it proves | What it does not prove |
|---|---|---|
| Model returns `INJECTION_TRIGGERED` | Behaviour was influenced. | Data access or tool execution. |
| Model prints a secret-looking value | Possible disclosure. | That the value is genuine or sensitive. |
| UI shows a proposed `create_ticket` call | Tool selection was generated. | That the ticket was created. |
| Server log records a successful API response | Tool execution occurred. | The full business impact without checking state. |
| New ticket exists with attacker-controlled content | Confirmed state change. | Access to unrelated private data. |
| Database record changed | Confirmed backend impact. | Broader compromise unless further evidence exists. |

## Facts, assumptions, and impact

A useful finding format:

### Facts

- the input source was attacker-controlled,
- the content entered the context,
- the model changed behaviour,
- a specific tool call was executed,
- a specific object or record changed.

### Assumptions

- the model may be able to access other documents,
- a tool may have broader permissions,
- the same attack may affect other users.

Assumptions should be tested or clearly labelled.

### Possible impact

What could happen if the architecture and permissions allow it.

### Confirmed impact

What the test actually demonstrated.

## Severity factors

Consider:

- sensitivity of accessible data,
- tool permissions,
- current-user authorization enforcement,
- need for attacker or victim interaction,
- repeatability,
- approval requirements,
- action reversibility,
- affected user population,
- monitoring and detection,
- whether the finding is output-only or produces a verified state change.

## Reporting rule

Do not report a generated sentence as a completed transaction, sent email, executed command, or data breach without downstream evidence.
