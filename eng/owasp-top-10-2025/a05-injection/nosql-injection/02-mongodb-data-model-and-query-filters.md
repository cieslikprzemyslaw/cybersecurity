# MongoDB Data Model and Query Filters

## Documents, collections, and databases

MongoDB stores records as documents containing key-value pairs.

```json
{
  "username": "admin",
  "email": "admin@example.com",
  "role": "administrator"
}
```

Documents with a similar purpose are grouped into collections. Collections are grouped into databases.

## Filters are structured objects

A MongoDB filter is commonly represented as an object or associative array.

PHP-style examples are used here because they clearly show nested structures.

### Simple equality

```php
['username' => 'admin']
```

Meaning:

```text
username equals admin
```

### Multiple fields

```php
[
  'username' => 'admin',
  'password' => 'secret'
]
```

Meaning:

```text
username equals admin
AND
password equals secret
```

Multiple top-level field conditions normally create an implicit `AND`.

## Nested operator objects

A field can map to a nested object containing a query operator.

### Less than

```php
['age' => ['$lt' => '50']]
```

Meaning:

```text
age is less than 50
```

### Not equal

```php
['password' => ['$ne' => 'test']]
```

Meaning:

```text
password is not equal to test
```

### Not in a list

```php
['username' => ['$nin' => ['admin', 'jude']]]
```

Meaning:

```text
username is not admin
AND
username is not jude
```

### Combined filter

```php
[
  'username' => ['$nin' => ['admin', 'jude']],
  'password' => ['$ne' => 'aweasdf']
]
```

Meaning:

```text
Find a document where:
- username is not admin or jude,
- AND password is not aweasdf.
```

If an application treats the first returned document as a successful login, this type of filter manipulation can produce authentication bypass.

## Regex filters

```php
['password' => ['$regex' => '^.{7}$']]
```

Meaning:

```text
Does the entire password contain exactly seven characters?
```

Regex parts:

```text
^      start of string
.      any character
{7}    exactly seven repetitions
$      end of string
```

A prefix test may look like:

```php
['password' => ['$regex' => '^c....$']]
```

Meaning:

```text
Does a five-character password begin with c?
```

## Why request parsing matters

The attacker may not need to send literal PHP or JavaScript code. The web framework may convert request notation into a nested object.

Example URL-encoded body:

```http
user[$ne]=xxxx&pass[$ne]=yyyy
```

A PHP-style parser may interpret it conceptually as:

```php
$user = ['$ne' => 'xxxx'];
$pass = ['$ne' => 'yyyy'];
```

A filter that was intended to be:

```php
[
  'username' => $user,
  'password' => $pass
]
```

may become:

```php
[
  'username' => ['$ne' => 'xxxx'],
  'password' => ['$ne' => 'yyyy']
]
```

The key vulnerability is the type change:

```text
expected string -> received nested operator object
```

## JSON example

Expected input:

```json
{
  "username": "admin"
}
```

Dangerous accepted shape:

```json
{
  "username": {
    "$ne": null
  }
}
```

The exact behaviour depends on the framework, parser, database driver, and query construction.

## Important operator reminder

Useful operators for understanding MongoDB queries include:

- `$eq` - equal
- `$ne` - not equal
- `$gt` - greater than
- `$gte` - greater than or equal
- `$lt` - less than
- `$lte` - less than or equal
- `$in` - in a list
- `$nin` - not in a list
- `$regex` - regular-expression matching

The official MongoDB [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/) reference is important because operators and supported behaviour should be checked against the actual database version and query context.

## AppSec takeaway

Do not memorise operators without understanding the object.

Read a filter in this order:

1. Identify the field.
2. Check whether the value is primitive or nested.
3. Identify the nested operator.
4. Translate the filter into plain language.
5. Ask whether the client should have been allowed to control that structure.
