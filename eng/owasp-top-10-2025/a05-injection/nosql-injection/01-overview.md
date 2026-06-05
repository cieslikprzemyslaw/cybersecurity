# NoSQL Injection Overview

## What is NoSQL Injection?

NoSQL Injection occurs when untrusted input changes the meaning of a query executed by a NoSQL database or query engine.

NoSQL is a broad term. Different NoSQL databases use different storage models, query languages, APIs, and operators. The exact payload syntax therefore depends on the database, language, framework, driver, and request parser.

This section focuses mainly on MongoDB-style examples.

## MongoDB data model

MongoDB stores data in documents rather than relational rows.

A simplified document may look like:

```json
{
  "_id": "example-id",
  "username": "lphillips",
  "first_name": "Logan",
  "last_name": "Phillips",
  "age": "65",
  "email": "lphillips@example.com"
}
```

Related documents are grouped into collections. Collections are grouped into databases.

A rough comparison is:

```text
Relational database     MongoDB
-------------------     -------
row / record            document
table                   collection
database                database
WHERE condition         query filter
```

The comparison is useful for orientation, but MongoDB documents and relational rows are not identical data structures.

## Main NoSQL Injection forms

### Operator Injection

Operator Injection happens when attacker-controlled input is interpreted as a query operator or nested query object rather than a simple value.

Example mental change:

```text
Expected:
username equals supplied string

Unsafe result:
username is not equal to supplied string
```

### Syntax Injection

Syntax Injection happens when input breaks out of a query expression and adds new syntax.

This can resemble SQL Injection, but the syntax is specific to the affected NoSQL query mechanism. In MongoDB, one risky example is user input concatenated into a custom JavaScript expression through `$where`.

Syntax Injection is generally less common than Operator Injection because it requires unsafe custom expression construction.

## SQL Injection vs NoSQL Injection

### SQL Injection

```text
user input -> SQL string -> SQL parser -> changed SQL query
```

Typical focus:

- quotes,
- comments,
- UNION,
- subqueries,
- SQL-specific functions.

### NoSQL Injection

```text
user input -> value/object/operator/expression -> NoSQL query engine -> changed query
```

Typical focus:

- nested JSON or associative arrays,
- parser behaviour,
- operator-shaped input,
- dynamic objects,
- custom JavaScript expressions,
- true/false response differences.

NoSQL Injection is not simply SQL Injection with different keywords. The request data structure often matters as much as the visible text.

## Common attack surfaces

- login forms,
- user lookup endpoints,
- search and filter APIs,
- product category filters,
- profile endpoints,
- JSON APIs,
- URL-encoded form bodies,
- query-string parameters,
- background fetch/XHR requests,
- GraphQL or API resolver filters,
- administrative search functions.

## Possible impact

- authentication bypass,
- unauthorised record access,
- user enumeration,
- disclosure of sensitive fields,
- blind extraction of secrets,
- administrator account compromise,
- filtering bypass,
- unreleased or hidden data exposure,
- verbose error and technology disclosure.

## Root causes

- accepting objects where strings are expected,
- missing schema validation,
- missing type enforcement,
- placing request objects directly into database filters,
- allowing clients to select query operators,
- unsafe dynamic query construction,
- concatenating input into `$where` or custom JavaScript,
- querying plaintext passwords,
- trusting frontend-generated values,
- exposing different responses that create a reliable oracle.

## Core AppSec questions

1. What input do I control?
2. What data type does the backend receive?
3. What parser processes the request?
4. Is input treated as a value or query structure?
5. Which database/query system receives it?
6. What response proves the query changed?
7. What is fact and what is only assumption?
8. What secure implementation would keep query structure server-controlled?
