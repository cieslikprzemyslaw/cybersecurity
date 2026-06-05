# NoSQL Injection Regression Tests

## Purpose

These tests should prove that untrusted input cannot become a NoSQL operator, nested query object, custom JavaScript expression, or secret-dependent oracle.

## Authentication tests

- Valid username and password succeed.
- Invalid username and password fail.
- Valid username with invalid password fails.
- Invalid username with valid-looking password fails.
- Arrays for username or password are rejected.
- Objects for username or password are rejected.
- Nested operator objects are rejected.
- Authentication does not succeed merely because a query returns a document.
- Passwords are verified against hashes, not queried as plaintext.

## JSON body tests

Reject:

```json
{
  "username": {
    "$ne": null
  },
  "password": "test"
}
```

Reject:

```json
{
  "username": "admin",
  "password": {
    "$regex": ".*"
  }
}
```

Reject:

```json
{
  "$where": "true"
}
```

Expected result:

- controlled `400` or validation response,
- no database error,
- no authentication,
- no user record returned.

## URL-encoded form tests

Test bracket notation where primitive strings are expected:

```http
username[$ne]=x&password[$ne]=y
```

Expected result:

- rejected as invalid structure,
- not converted into a database operator filter,
- no session created.

## User lookup tests

- Exact valid username returns only that authorised record.
- Unknown username returns a controlled not-found result.
- A quote does not trigger an interpreter error.
- JavaScript boolean syntax does not broaden the query.
- An always-true expression does not return another user.
- User lookup cannot expose privileged fields unnecessarily.
- The endpoint enforces authorisation separately from lookup logic.

## Search and category tests

- Known category returns only intended records.
- Unknown category returns no results or validation error.
- A quote is treated as data or rejected safely.
- Boolean expressions do not return all records.
- Hidden or unreleased records remain excluded by server-side policy.
- Filtering cannot override mandatory visibility conditions.

## Oracle-resistance tests

For conditions about secret data:

```text
condition true
condition false
```

confirm that:

- the endpoint does not evaluate attacker-controlled conditions,
- response bodies do not reveal condition truth,
- response lengths do not expose secret-dependent differences,
- error behaviour is controlled,
- rate limits detect repeated probing.

## Error handling tests

The response must not expose:

- `MongoServerError`,
- `OperationFailure`,
- `JSInterpreterFailure`,
- stack traces,
- internal query strings,
- database host/port,
- source file paths.

## Code review tests

- No request object is passed directly to a database query.
- No object spread or merge introduces client keys into filters.
- Query fields and operators are selected by trusted server code.
- Credential inputs are checked with primitive type guards.
- Request schemas reject unknown and nested fields where unnecessary.
- `$where` and custom JavaScript are absent unless formally justified and isolated.
- Raw database filters are not accepted from public clients.

## Example test intentions

```text
Given username is an object
When login is submitted
Then return validation error
And do not query MongoDB
```

```text
Given category contains quote and boolean syntax
When product filtering is requested
Then treat it as an invalid category/value
And do not broaden product visibility
```

```text
Given repeated character-position probes
When request volume exceeds the expected pattern
Then apply rate limiting and create a security signal
```

## Fix verification

A NoSQL Injection fix is complete only when:

- the original exploit no longer changes query behaviour,
- invalid structures are rejected before database access,
- query structure remains server-controlled,
- sensitive data is not exposed,
- no reliable true/false oracle remains,
- automated regression tests are committed.
