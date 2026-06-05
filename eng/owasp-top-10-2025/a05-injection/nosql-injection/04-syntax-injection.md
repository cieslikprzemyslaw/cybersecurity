# NoSQL Syntax Injection

## Definition

NoSQL Syntax Injection occurs when user input is inserted into a query expression in a way that allows the input to break out and add new syntax.

This resembles SQL Injection conceptually, but the syntax depends on the affected NoSQL query mechanism.

## MongoDB `$where` example

A risky custom JavaScript-style query may look like:

```python
mycol.find({
    "$where": "this.username == '" + username + "'"
})
```

The input is concatenated into an expression.

MongoDB lists `$where` in its official [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/) reference. In AppSec notes, treat it as a feature to review carefully because it evaluates JavaScript-style expressions.

A normal username:

```text
admin
```

may create:

```javascript
this.username == 'admin'
```

A quote may break the expression:

```text
admin'
```

The resulting error can indicate that input reached executable query syntax.

## Detection through errors

A quote may cause:

- a MongoDB operation failure,
- a JavaScript interpreter failure,
- a syntax error,
- HTTP 500,
- a controlled application error.

An error is useful evidence, but it is not complete proof by itself. A controlled true/false comparison is stronger.

## True and false conditions

Conceptual false test:

```javascript
admin' && 0 && 'x
```

Conceptual true test:

```javascript
admin' && 1 && 'x
```

If the output differs predictably, the injected condition is affecting query logic.

## Always-true condition

A lab payload such as:

```text
'||1||'
```

uses JavaScript logical OR.

```javascript
||
```

means logical OR.

The number `1` is truthy in JavaScript, so an expression conceptually similar to:

```javascript
this.category == 'Gifts' || 1 || ''
```

evaluates as true for every document.

This may:

- return all products,
- bypass a category filter,
- return the first user,
- expose records outside the intended condition.

## Why an unexpected administrator may appear

An always-true lookup may match multiple documents. If the application or driver returns one document and no explicit ordering is enforced, the response may contain the first matching record.

If that record is the administrator, it does not necessarily mean the session changed. It means the vulnerable lookup returned the administrator document.

## Why Syntax Injection is less common

This type of issue usually requires developers to use:

- `$where`,
- custom JavaScript expressions,
- unsafe string-built query logic,
- another query feature that interprets constructed syntax.

Built-in structured filters such as:

```python
{"username": username}
```

avoid this specific string-expression pattern when `username` is also strictly validated as a primitive value.

Structured filters can still be vulnerable to Operator Injection if unsafe objects are accepted.

## Syntax Injection vs Operator Injection

| Question | Syntax Injection | Operator Injection |
|---|---|---|
| Does input break out of an expression? | Usually yes | Not required |
| Main shape | String/expression syntax | Nested object/operator |
| Example mechanism | `$where` JavaScript | `$ne`, `$nin`, `$regex` |
| Common root cause | String-built custom query | Missing type/schema enforcement |
| Relative frequency | Usually rarer | Often more realistic |

## Remediation

- Avoid `$where` and custom JavaScript queries where possible.
- Never concatenate untrusted input into executable expressions.
- Prefer built-in structured filters.
- Enforce primitive input types.
- Allowlist server-supported fields and operations.
- Disable verbose production errors.
- Add tests using quotes, boolean expressions, and encoded equivalents.

## Testing reminder

A single quote is a probe, not a conclusion.

The stronger evidence chain is:

```text
baseline
-> syntax-sensitive error
-> controlled true condition
-> controlled false condition
-> predictable query-result difference
```
