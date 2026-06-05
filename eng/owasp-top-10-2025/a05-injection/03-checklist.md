# A05: Injection Checklist

Use this checklist during learning, code review, or authorised testing.

## General injection questions

- Does user-controlled input reach an interpreter or processing engine?
- Can the input change the structure of a query, command, template, expression, or instruction?
- Is the input treated as data, or can it become syntax?
- What exact data structure reaches the backend?
- Is it a string, array, object, nested object, JSON value, form value, query parameter, cookie, or header?
- Is validation being used as the only defence?
- Is the secure implementation based on safe APIs, parameterization, strict schemas, type enforcement, or allowlisted operations?

## SQL Injection checks

- Are SQL queries built by concatenating strings with user input?
- Are URL parameters, request body values, cookies, or headers used in SQL queries?
- Does adding a single quote change the response or cause an error?
- Does adding a SQL comment repair a broken query?
- Can true and false conditions change the response behaviour?
- Can `UNION SELECT` return additional data in the response?
- Can blind SQLi be detected through response differences or timing?
- Are database errors exposed to the user?

## NoSQL Injection checks

### Input and data structure

- Does the endpoint expect a primitive string but accept an object or array?
- Can JSON input contain nested objects where a string is expected?
- Can URL-encoded form notation such as `field[operator]=value` become a nested server-side object?
- Does a body parser preserve attacker-controlled keys beginning with `$`?
- Can a user control query field names, operators, sorting, projection, or filters?

### Operator Injection

- Can input become an operator object such as `$ne`, `$nin`, `$gt`, or `$regex`?
- Does changing a value into a nested object alter authentication or lookup behaviour?
- Can the query return a user without validating the supplied credential?
- Can regex conditions create a true/false oracle for secret extraction?

### Syntax Injection

- Is user input concatenated into `$where` or another custom JavaScript expression?
- Does a single quote cause an error or change behaviour?
- Do controlled true and false conditions produce different responses?
- Can an always-true condition return records outside the intended filter?
- Are verbose MongoDB or JavaScript interpreter errors exposed?

### Attack-surface discovery

- Is the visible browser URL the real data endpoint?
- Does the page make background fetch/XHR requests?
- Is the vulnerable request visible only in Burp HTTP history or browser DevTools?
- Are redirects hiding the endpoint that actually processes the data?

## Evidence to look for

- database or interpreter error messages,
- HTTP 500 or controlled error after syntax-sensitive input,
- different response body for true vs false conditions,
- a lookup returning a different user,
- additional products or records appearing,
- a response-length or marker difference,
- a secret length confirmed through repeated conditions,
- one successful character result per secret position.

## Secure implementation checks

- Prepared statements or parameterized APIs are used for SQL.
- NoSQL inputs are validated against strict schemas.
- Authentication fields are enforced as primitive strings.
- Nested objects and operator-shaped input are rejected where not required.
- User-controlled values are not inserted into `$where` or custom JavaScript.
- Built-in structured query filters are preferred over custom query expressions.
- Passwords are verified using password-hash verification, not queried as plaintext values.
- Database users follow least privilege.
- Production does not expose verbose database or interpreter errors.
- Rate limiting and monitoring make automated extraction harder.
- Regression tests cover both string payloads and object-shaped payloads.

## Frontend/AppSec angle

- Do not assume values are safe because the frontend generated the link or form.
- Do not assume hidden fields, cookies, predefined categories, or client validation are trusted.
- Do not rely on GET vs POST for security.
- Inspect background API requests, not only the address bar.
- The security boundary is how the backend parses input and constructs the query.

For longer topic-specific checklists, use:

- [SQL Injection cheat sheet](sql-injection/cheat-sheet.md)
- [NoSQL Injection cheat sheet](nosql-injection/cheat-sheet.md)
