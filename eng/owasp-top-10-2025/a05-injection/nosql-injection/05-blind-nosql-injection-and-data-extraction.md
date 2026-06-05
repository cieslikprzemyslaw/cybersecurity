# Blind NoSQL Injection and Data Extraction

## What makes extraction blind?

In a blind scenario, the application does not directly display the secret.

Instead, the attacker asks a series of questions and observes whether each condition is true or false.

```text
secret not displayed
-> condition about secret
-> response difference
-> one bit of information learned
```

The application becomes an oracle.

## `$regex` operator approach

A MongoDB-style filter may test password length:

```php
['password' => ['$regex' => '^.{7}$']]
```

Question:

```text
Does the entire password contain exactly seven characters?
```

A prefix test may look like:

```php
['password' => ['$regex' => '^c....$']]
```

Question:

```text
Does a five-character password begin with c?
```

The tester repeats controlled questions until the correct length and characters are found.

## Syntax Injection approach

When the injection point is a JavaScript-style expression rather than an operator object, the condition may refer directly to a document field.

### Length test

```javascript
this.password.length == 8
```

Question:

```text
Does the current document's password have eight characters?
```

### Character test

```javascript
this.password[0] == 'a'
```

Question:

```text
Is the first password character a?
```

JavaScript strings use zero-based indexing:

```text
first character  -> index 0
second character -> index 1
...
eighth character -> index 7
```

## Building a reliable oracle

Before automation, manually establish:

1. A known true condition.
2. A known false condition.
3. A stable response difference.
4. A marker that can be detected automatically.

Possible markers:

- user object returned,
- `"username": "administrator"` present,
- specific response length,
- `Could not find user`,
- a different JSON message,
- status or redirect difference.

The marker must be repeatable and caused by the condition, not by noise.

## Password-length workflow

Conceptually:

```text
length == 1 -> false
length == 2 -> false
...
length == 8 -> true
```

Once the length is known, character extraction can begin.

## Character extraction workflow

For every index:

```text
index 0: test a-z
index 1: test a-z
...
index 7: test a-z
```

The correct character is the request that produces the true response marker.

## Burp Intruder

### Cluster Bomb

Cluster Bomb tests every combination of multiple payload sets.

For:

```text
positions: 0-7
letters:   a-z
```

it tests combinations such as:

```text
0+a, 0+b, 0+c ...
1+a, 1+b, 1+c ...
```

This is appropriate when every letter must be tested at every position.

### Pitchfork

Pitchfork pairs payload values in parallel:

```text
0+a
1+b
2+c
```

This would not test every letter for every index, so it is not appropriate for the same extraction plan.

## Encoding lesson

When the payload is placed in a URL:

- spaces may be encoded as `%20`,
- `&&` must not be allowed to split query parameters,
- `&` is encoded as `%26`,
- `[` and `]` may be encoded as `%5B` and `%5D`,
- quotes and comparison operators may also require encoding.

Incorrect `%u...` encoding or repeated encoding can cause the server to receive literal text instead of the intended expression.

## Security impact

A blind oracle can disclose:

- usernames,
- email addresses,
- internal flags,
- tokens,
- roles,
- secrets,
- credentials when stored or queried unsafely.

Password extraction is especially serious if plaintext or reversible password data is queryable. Secure applications should store password hashes and verify them outside the lookup query.

## Defensive lesson

- Do not allow user-controlled operators or expressions.
- Enforce request schemas and primitive types.
- Avoid custom JavaScript queries.
- Hash passwords securely.
- Return controlled responses.
- Rate-limit repeated probes.
- Monitor unusual query patterns and high-volume character testing.
