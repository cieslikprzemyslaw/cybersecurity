# SSTI Practical Work Summary

## Completed exercises

| Platform | Engine | Runtime | Main proof | Demonstrated deeper impact |
|---|---|---|---|---|
| TryHackMe | Smarty | PHP | Template modifier/expression changed output | PHP function reached command execution in the lab |
| TryHackMe | Pug | Node.js | JavaScript expression returned a result | Node process API reached command execution |
| TryHackMe | Jinja2 | Python | Template expression returned a result | Python object traversal reached `subprocess` |
| PortSwigger | ERB | Ruby | Arithmetic expression returned `49` | Ruby `File.delete()` deleted the specified file |

## Shared mental model

```text
controlled input
  -> template engine evaluates syntax
  -> runtime objects or APIs may become reachable
  -> impact depends on context, configuration, and privileges
```

## Most useful cross-language lesson

The language and API differ, but the AppSec reasoning stays consistent:

1. Find the controlled input.
2. Find the rendering location.
3. Separate reflection from evaluation.
4. Identify the engine with evidence.
5. Prove one capability at a time.
6. Do not label evaluation as full RCE without evidence.
7. Keep root cause separate from impact.
8. Fix the data/template-source boundary.

## Why one PortSwigger lab is sufficient now

The PortSwigger lab added a full black-box reasoning flow and a fourth engine/runtime after three TryHackMe practical exercises.

Current coverage includes:

- multiple template syntaxes,
- multiple server languages,
- reflection versus evaluation,
- fingerprinting concepts,
- direct runtime APIs,
- command/process API differences,
- root cause,
- remediation,
- regression testing.

More labs would add depth, but they are not required before continuing the current A05 roadmap.
