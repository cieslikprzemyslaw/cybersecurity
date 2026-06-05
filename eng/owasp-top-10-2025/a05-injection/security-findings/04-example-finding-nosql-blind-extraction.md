# Example Security Finding: NoSQL Syntax Injection Allows Blind Extraction of Administrator Credentials

> Example finding based on an authorised training lab. Hostnames, session values, and credentials are omitted.

## Severity

**High**

## Summary

A user lookup endpoint constructs a MongoDB query using user-controlled input inside a JavaScript-style expression.

An attacker can break out of the intended username value, add boolean conditions, and observe different responses depending on whether the injected condition is true or false.

This behaviour creates a boolean oracle that can be used to determine sensitive field values one character at a time. In the training scenario, the administrator password was recovered and used to access the administrator account.

## Affected feature

```http
GET /user/lookup?user=<username>
```

Affected parameter:

```text
user
```

## Evidence

### Baseline

A normal lookup returned the requested user:

```json
{
  "username": "wiener",
  "email": "wiener@normal-user.net",
  "role": "user"
}
```

### Syntax-sensitive input

Adding a quote changed the response to:

```json
{
  "message": "There was an error getting user details"
}
```

This indicated that the parameter affected query syntax.

### Always-true condition

A JavaScript-style always-true condition caused the endpoint to return the administrator record instead of the requested normal user.

### Boolean oracle

A condition checking password length returned the administrator record only when the tested length was correct.

Character-position conditions produced the same true/false response difference, allowing the password to be reconstructed.

## Attack path

```text
controlled user parameter
-> break out of query expression
-> inject boolean condition
-> identify true/false response marker
-> determine password length
-> test each character position
-> recover administrator password
-> administrator account compromise
```

## Impact

An unauthenticated or low-privileged attacker with access to the endpoint may be able to:

- retrieve records outside the intended lookup,
- enumerate privileged accounts,
- infer sensitive field values,
- extract credentials or tokens,
- compromise administrator accounts,
- access administrator-only functionality.

The impact depends on the fields available to the query, the endpoint's authentication requirements, and whether sensitive values are stored in a queryable form.

## Root cause

The application allows user input to become part of executable MongoDB/JavaScript query syntax.

Likely contributing issues include:

- unsafe use of `$where` or custom JavaScript query logic,
- string concatenation during query construction,
- missing strict input validation,
- sensitive password data being queryable,
- distinct responses creating a stable boolean oracle,
- insufficient rate limiting.

## Remediation

### Query construction

- Remove custom JavaScript query construction where possible.
- Do not concatenate user input into `$where` or any executable expression.
- Use built-in structured filters with server-selected fields and operators.
- Enforce that `user` is a primitive string.

### Authentication and password storage

- Store passwords using an appropriate password-hashing algorithm.
- Query users only by an exact validated username or email.
- Verify the submitted password against the stored hash outside the database query.
- Never query plaintext password values.

### Response and abuse controls

- Return controlled, non-verbose errors.
- Avoid response differences that reveal secret-dependent condition results.
- Rate-limit repeated lookup attempts.
- Monitor for quotes, JavaScript operators, repeated length tests, and character-position probing.

## Verification steps

After remediation, confirm:

- a quote does not cause an interpreter or application error,
- JavaScript expressions are treated as data or rejected,
- an always-true condition does not return another user,
- password length and character conditions do not affect responses,
- nested or non-string values are rejected,
- sensitive password data is not queryable,
- rate limiting and monitoring detect automated probing.

## Suggested regression tests

```text
Given user contains a quote
When lookup is requested
Then return a controlled validation/not-found response
And do not expose database errors
```

```text
Given user contains JavaScript boolean syntax
When lookup is requested
Then do not broaden the result set
And do not return another user's record
```

```text
Given repeated secret-dependent conditions
When requests are sent
Then responses do not reveal condition truth
And abuse controls are triggered
```
