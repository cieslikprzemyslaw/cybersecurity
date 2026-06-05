# Lab: Detecting NoSQL Injection

## Source

PortSwigger Web Security Academy: Detecting NoSQL injection

## Goal

Exploit a vulnerable product category filter so that the application displays products that should not be returned by the selected category, including unreleased products.

## Vulnerable feature

The vulnerable feature was the product category filter.

Controlled input:

```http
GET /filter?category=Gifts
```

The `category` query parameter appeared to be inserted into a MongoDB JavaScript-style query expression.

## Baseline

Normal request:

```http
GET /filter?category=Gifts
```

Observed result:

- `200 OK`
- page heading: `Gifts`
- three visible products

This baseline was important because later responses needed to be compared against known normal behaviour.

## Testing flow

### 1. Quote test

Request concept:

```http
GET /filter?category=Gifts'
```

Observed result:

- `500 Internal Server Error`
- `JSInterpreterFailure`
- JavaScript syntax error
- MongoDB service information was exposed

Interpretation:

The quote likely broke a JavaScript-style expression used by the MongoDB query.

This suggested Syntax Injection rather than Operator Injection.

### 2. Always-true condition

Request concept:

```text
Gifts'||1||'
```

Why it was tested:

```javascript
||
```

is logical OR, and `1` is truthy in JavaScript.

The resulting backend expression was conceptually similar to:

```javascript
this.category == 'Gifts' || 1 || ''
```

Observed result:

- `200 OK`
- the response contained many products rather than only the three `Gifts` products
- products outside the intended category were returned

## Evidence summary

```text
Normal category
-> 200 OK
-> 3 Gifts products

Category + quote
-> 500
-> JSInterpreterFailure / syntax error

Category + always-true condition
-> 200 OK
-> broadened product set
```

## What this proved

- `category` was user-controlled.
- The input affected JavaScript-style query syntax.
- The query result changed predictably.
- The category restriction could be bypassed.
- Hidden or unreleased products could be exposed.

## Vulnerability classification

```text
NoSQL Syntax Injection
```

This was not Operator Injection because the test did not send a nested `$ne` or `$regex` query object. It broke out of a string/expression and inserted JavaScript logic.

## Root cause

The application likely concatenated the category value into a MongoDB custom JavaScript expression instead of using a safe structured filter with a validated category value.

## Real impact

The vulnerability allowed a user to bypass product filtering and expose records outside the intended result set.

In a real application, similar behaviour could expose:

- hidden products,
- unreleased content,
- archived records,
- internal-only data,
- records belonging to other users.

## Developer remediation

- Do not concatenate category values into `$where` or custom JavaScript.
- Use a server-controlled structured query.
- Validate the category against an allowlist of known category identifiers.
- Keep mandatory visibility conditions separate from client-controlled filtering.
- Do not expose raw database or interpreter errors.
- Add regression tests for quote and boolean-expression input.

## Regression tests

- A quote in `category` must not cause a database error.
- JavaScript operators in `category` must not change the number of returned products.
- Unknown categories must return no results or a controlled validation response.
- Unreleased products must remain excluded regardless of category input.
- MongoDB and JavaScript errors must not appear in the response.

## AppSec lesson

The strong evidence was not the quote alone.

The complete evidence chain was:

```text
baseline
-> syntax error
-> valid always-true condition
-> broadened dataset
```

## Status

**PASS**
