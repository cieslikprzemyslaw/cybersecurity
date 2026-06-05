# NoSQL Injection Lab Summary

## Completed labs

1. Detecting NoSQL injection
2. Exploiting NoSQL injection to extract data

## Comparison

| Area | Lab 1 | Lab 2 |
|---|---|---|
| Feature | Product category filter | Background user lookup API |
| Controlled input | `category` | `user` |
| Injection form | Syntax Injection | Syntax Injection |
| First evidence | Quote caused JS interpreter error | Quote caused lookup error |
| Confirmation | Always-true condition broadened products | Always-true condition returned administrator |
| Main impact | Hidden/unreleased product exposure | Blind credential extraction and account compromise |
| Oracle required | No | Yes |
| Automation | Not required | Burp Intruder |
| Key lesson | Query logic can bypass filters | Response differences can leak secrets |

## Shared evidence model

Both labs followed:

```text
1. Establish baseline.
2. Change one input.
3. Observe syntax-sensitive behaviour.
4. Test a controlled logical condition.
5. Compare results.
6. Demonstrate impact.
7. Explain root cause and remediation.
```

## Most important difference

The first lab returned the impact directly by showing a broader product set.

The second lab did not display the password. It returned a true/false signal, so the secret had to be reconstructed through repeated conditions.

## Personal learning outcome

The largest practical lesson was attack-surface discovery.

The vulnerable function may be:

- a background API request,
- a filter endpoint,
- a cookie,
- a body value,
- or another request that is not obvious from the visible page URL.

NoSQL Injection testing must consider both the text of the input and the data structure/parser that processes it.
