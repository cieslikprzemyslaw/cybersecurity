# NoSQL Operator Injection

## Definition

Operator Injection occurs when user-controlled input is interpreted as a NoSQL query operator or nested query object.

The attacker may not need to break the query syntax. Instead, they change the meaning of a field condition.

## Vulnerable authentication pattern

Simplified PHP/MongoDB example:

```php
$user = $_POST['user'];
$pass = $_POST['pass'];

$query = new MongoDB\Driver\Query([
    'username' => $user,
    'password' => $pass
]);
```

The developer assumes:

```text
$user is a string
$pass is a string
```

But the request parser may produce arrays or nested objects.

## Authentication bypass example

Request body:

```http
user[$ne]=xxxx&pass[$ne]=yyyy
```

Conceptual server-side values:

```php
$user = ['$ne' => 'xxxx'];
$pass = ['$ne' => 'yyyy'];
```

Resulting filter:

```php
[
  'username' => ['$ne' => 'xxxx'],
  'password' => ['$ne' => 'yyyy']
]
```

Plain-language meaning:

```text
Return documents where:
- username is not xxxx,
- AND password is not yyyy.
```

This condition may match many users.

If the application assumes that any returned document proves valid authentication, it may create a session for the first returned user.

## Trust assumption

The unsafe trust assumption is:

```text
The client will submit simple strings.
```

The server must not rely on the form control or frontend to preserve the type.

## Other useful operators

Use the official MongoDB [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/) reference when checking operator names, supported behaviour, and version-specific details.

### `$nin`

```php
['username' => ['$nin' => ['admin', 'jude']]]
```

This excludes selected usernames and may cause the query to return another account.

### `$regex`

```php
['password' => ['$regex' => '^a....$']]
```

This can ask whether a field matches a pattern. If the application response changes for a match, the endpoint may become a blind extraction oracle.

## Important request formats

Operator-shaped input may arrive through:

- JSON bodies,
- URL-encoded forms,
- query-string bracket notation,
- nested GraphQL variables,
- framework-specific object parsing,
- filters passed directly from client-side state.

Example JSON:

```json
{
  "username": {
    "$ne": "nobody"
  },
  "password": {
    "$ne": "invalid"
  }
}
```

## Evidence of Operator Injection

Evidence may include:

- login succeeds with invalid credentials,
- a different user is returned,
- the first collection document is used,
- object input behaves differently from string input,
- `$regex` conditions produce repeatable true/false responses,
- response content changes when the operator condition changes.

## Root cause

The root cause is broader than string concatenation.

Common causes include:

- missing primitive type checks,
- missing request schema validation,
- passing request objects directly to database APIs,
- allowing client-controlled operators,
- unsafe dynamic filter construction,
- authentication logic that trusts any returned document.

## Secure design

- Validate request bodies against a strict schema.
- Require `username` and `password` to be strings.
- Reject arrays and objects for credential fields.
- Build query structure on the server.
- Do not accept raw database filters from untrusted clients.
- Retrieve a user by an exact validated identifier.
- Verify the password using a secure password-hash verification function.
- Return generic authentication errors.
- Add rate limiting and monitoring.

## Testing reminder

Use only in legal labs or explicitly authorised environments.

Before testing an operator, explain:

```text
What input type does the endpoint expect?
What alternative type am I testing?
What query behaviour should change?
What response would prove it?
```
