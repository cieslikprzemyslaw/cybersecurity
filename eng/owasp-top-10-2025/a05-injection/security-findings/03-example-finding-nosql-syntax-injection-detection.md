# Example Security Finding: NoSQL Syntax Injection in Product Filter

> Example finding based on an authorised training lab. Hostnames, session values, and sensitive data are omitted.

## Severity

**Medium**

## Summary

A product filtering endpoint processes user-controlled input inside a MongoDB-style query expression.

Syntax-sensitive input changes the application response, and controlled true/false conditions produce different result sets. This indicates that the parameter can alter query logic rather than being treated only as data.

In the training scenario, an always-true condition caused the application to return records outside the intended filter.

## Affected feature

```http
GET /filter?category=<category>
```

Affected parameter:

```text
category
```

## Evidence

### Baseline

A normal category filter returned the expected products for the selected category.

### Syntax-sensitive input

Adding a single quote changed the response and indicated that the value affected query syntax.

This was useful evidence, but it was not enough on its own. The behaviour needed to be compared with controlled true and false conditions.

### True condition

A JavaScript-style condition that evaluated to true returned products outside the original category filter.

### False condition

A matching condition that evaluated to false removed or changed the expected product results.

The stable difference between true and false responses confirmed that user input could influence the backend query expression.

## Attack path

```text
controlled category parameter
-> input reaches query expression
-> quote confirms syntax-sensitive context
-> true condition changes result set
-> false condition changes result set differently
-> query filter bypass
```

## Impact

An attacker may be able to:

- bypass product or record filters,
- retrieve hidden or unreleased records,
- enumerate data outside the intended category,
- use response differences as a starting point for further NoSQL Injection testing.

The demonstrated impact is filter bypass and unintended data exposure. This finding does not claim credential extraction unless a separate oracle and sensitive field access are proven.

## Root cause

The application allows user input to become part of executable NoSQL query syntax.

Likely contributing issues include:

- unsafe custom query expression construction,
- string concatenation when building the filter,
- missing strict validation for category values,
- trusting frontend-generated category links,
- verbose or distinguishable responses for syntax and boolean tests.

## Remediation

- Do not concatenate user input into MongoDB `$where` or JavaScript-style query expressions.
- Build filters from server-selected fields and operators.
- Validate `category` against an allowlist of expected category identifiers.
- Treat unknown categories as no-result or controlled validation failures.
- Return controlled errors without exposing database or interpreter details.
- Add regression tests for quote, true-condition, and false-condition inputs.

## Verification steps

After remediation, confirm:

- a quote in `category` does not cause an interpreter error,
- true and false conditions are treated as data or rejected,
- always-true input does not broaden the result set,
- unknown categories do not expose hidden records,
- response differences no longer prove query-expression control.

## Suggested regression tests

```text
Given category contains a single quote
When the product filter is requested
Then return a controlled response
And do not expose database or JavaScript errors
```

```text
Given category contains an always-true expression
When the product filter is requested
Then do not return products outside the selected category
```

```text
Given category contains paired true and false conditions
When both requests are sent
Then responses do not reveal query-expression control
```
