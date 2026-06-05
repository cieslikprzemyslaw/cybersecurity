# NoSQL Injection Cheat Sheet

Use only in legal labs or authorised testing.

## Core mental model

```text
Expected:
field -> simple value

Dangerous:
field -> nested object/operator/expression
```

Ask:

```text
Is the input still data,
or did it become query logic?
```

## Before testing

- What feature am I testing?
- What exact request processes the data?
- Is there a background API request?
- What input do I control?
- What format is used?
- What type should the value be?
- What parser handles the input?
- What database/query mechanism may process it?
- What is my baseline?
- What result would prove changed query behaviour?

## Request formats to inspect

- query string,
- URL-encoded form body,
- JSON body,
- cookies,
- headers,
- GraphQL variables,
- fetch/XHR requests,
- hidden background API calls.

## MongoDB filter reading

```php
['username' => 'admin']
```

```text
username equals admin
```

```php
['username' => 'admin', 'role' => 'user']
```

```text
username equals admin AND role equals user
```

```php
['password' => ['$ne' => 'test']]
```

```text
password is not equal to test
```

```php
['username' => ['$nin' => ['admin', 'jude']]]
```

```text
username is not in the list admin/jude
```

## Operator reminder

| Operator | Meaning |
|---|---|
| `$eq` | equal |
| `$ne` | not equal |
| `$gt` | greater than |
| `$lt` | less than |
| `$in` | in list |
| `$nin` | not in list |
| `$regex` | regex match |

Check the official database documentation for exact behaviour.

## Operator Injection indicator

Expected:

```json
{"username":"admin"}
```

Unexpected accepted structure:

```json
{"username":{"$ne":null}}
```

Key question:

```text
Did the server enforce username as a string?
```

## Syntax Injection indicator

Evidence chain:

```text
normal input -> baseline
quote -> syntax/error change
true condition -> one result
false condition -> different result
```

A quote alone is not final proof.

## Always-true logic

In JavaScript-style expressions:

```javascript
||
```

means OR.

`1` is truthy, so a condition containing:

```javascript
|| 1 ||
```

may become true for every document.

## Boolean oracle

An oracle gives a repeatable true/false signal.

Example:

```text
TRUE  -> administrator JSON returned
FALSE -> user not found
```

Questions can then be asked about a secret:

```javascript
this.password.length == 8
this.password[0] == 'a'
```

## Regex reminder

```regex
^.{7}$
```

```text
exactly seven characters
```

```regex
^c....$
```

```text
five characters beginning with c
```

A regex string does nothing unless it is applied through `$regex`, `.match()`, or another actual regex operation.

## Intruder choice

### Cluster Bomb

Use when every value in payload set A must be combined with every value in payload set B.

Example:

```text
indexes 0-7 x letters a-z
```

### Pitchfork

Pairs payload values by position and does not test the full combination set.

## Encoding reminders

- `&` may split query parameters.
- Encode direct URL payloads correctly.
- Avoid accidental double encoding.
- `%uXXXX` is not normal URL percent-encoding for these requests.
- Confirm what the server actually receives.

## Evidence checklist

- Baseline response saved.
- Controlled input identified.
- Query-relevant endpoint confirmed.
- Facts separated from assumptions.
- True and false conditions compared.
- Stable marker identified.
- Sensitive result or broadened dataset demonstrated.
- Root cause described.
- Remediation proposed.
- Regression tests defined.

## Stop payload guessing

Before each test, write:

```text
What am I testing?
Why am I testing it?
What result do I expect?
What would a different result mean?
```

After each test:

```text
What changed?
What stayed the same?
What evidence did I get?
What does it suggest?
What is the next smallest controlled test?
```
