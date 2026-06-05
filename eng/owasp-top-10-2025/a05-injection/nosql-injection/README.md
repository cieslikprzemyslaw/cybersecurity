# NoSQL Injection

NoSQL Injection happens when untrusted input is used to construct a NoSQL query in a way that changes the intended query logic.

## Mental model

```text
user-controlled input
  -> primitive value, nested object, query operator, or expression
  -> NoSQL database/query engine
  -> changed query behaviour
```

The key question is:

```text
Did the application keep the input as data,
or did the input become part of the query?
```

## Main forms covered

### Operator Injection

The attacker causes an input that should be a simple value to become a nested query operator or object.

Examples include:

- `$ne`
- `$nin`
- `$gt`
- `$regex`

### Syntax Injection

The attacker breaks out of a query expression and adds new syntax. In MongoDB, this can occur when an application builds custom JavaScript-style expressions, for example through `$where`.

This is generally less common than Operator Injection because it depends on unsafe custom query construction.

## Files

- [01-overview.md](01-overview.md) - NoSQL Injection definition, mental model, impact, and root causes.
- [02-mongodb-data-model-and-query-filters.md](02-mongodb-data-model-and-query-filters.md) - MongoDB documents, collections, filters, and operators.
- [03-operator-injection.md](03-operator-injection.md) - object/operator manipulation and authentication bypass.
- [04-syntax-injection.md](04-syntax-injection.md) - custom JavaScript expressions, quote tests, and always-true conditions.
- [05-blind-nosql-injection-and-data-extraction.md](05-blind-nosql-injection-and-data-extraction.md) - boolean oracles and character extraction.
- [06-remediation.md](06-remediation.md) - developer remediation model.
- [cheat-sheet.md](cheat-sheet.md) - practical testing and code-review reminders.
- [regression-tests.md](regression-tests.md) - NoSQL-specific regression ideas.
- [learning-summary.md](learning-summary.md) - personal learning process and takeaways.

## Completed labs

- [Detecting NoSQL injection](labs/01-detecting-nosql-injection.md)
- [Exploiting NoSQL injection to extract data](labs/02-extracting-data-with-a-boolean-oracle.md)
- [Lab comparison](labs/summary.md)

## Supporting references

- TryHackMe: NoSQL Injection
- PortSwigger Web Security Academy: NoSQL injection
- MongoDB Manual: [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/)

## Key takeaway

The main defence is not a blacklist of `$` characters or quotes. The application must enforce expected input types and schemas, keep client input out of query structure, avoid custom JavaScript query construction, and verify passwords outside the database query using secure password-hash verification.
