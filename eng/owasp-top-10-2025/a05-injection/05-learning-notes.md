# A05: Injection - Learning Notes

## Wider mental model

SQL Injection and NoSQL Injection both helped me understand the wider A05 model:

```text
controlled input -> interpreter/query engine -> changed behaviour
```

The payload is not the root cause by itself. The vulnerability exists because the application allows untrusted input to become query syntax, a query operator, or part of an executable expression.

## SQL Injection lessons

- A single quote can break a SQL query when input is inserted into a string context.
- Adding a SQL comment can repair a query by commenting out the remaining SQL.
- `UNION SELECT` requires a compatible number of columns.
- `NULL` is useful when testing column count.
- Blind SQLi requires a reliable true/false oracle rather than directly displayed data.

## NoSQL Injection lessons

### NoSQL is not one query language

Different NoSQL databases use different data models and query systems. In MongoDB, queries commonly use structured objects or associative arrays.

### A value can become query logic

The important question is:

```text
Is my input still a simple value,
or has it become a nested query object/operator?
```

A login field expected to contain a string may be dangerous if the backend accepts a structure such as:

```php
['$ne' => 'test']
```

### Operator Injection and Syntax Injection are different

Operator Injection changes the query through structured operators such as `$ne`, `$nin`, or `$regex`.

Syntax Injection breaks out of a query expression and adds new syntax. In MongoDB this is less common and may involve custom JavaScript-style query logic such as `$where`.

### Background endpoints matter

In the data-extraction lab, the visible page was:

```text
/my-account?id=wiener
```

The vulnerable data request was a background request:

```text
/user/lookup?user=wiener
```

This reinforced that AppSec testing must inspect HTTP history, fetch/XHR traffic, and API calls rather than relying only on the browser address bar.

### Blind extraction is an oracle problem

The endpoint did not directly return the password. It revealed whether a statement about the password was true.

Examples of the questions asked conceptually:

```text
Does the administrator password have length 8?
Is character at index 0 equal to a?
Is character at index 1 equal to b?
```

A matching response meant true. A different response meant false.

## Encoding lesson

Raw payloads and encoded payloads are not interchangeable.

- Raw payload may be appropriate in a UI that performs encoding.
- Encoded characters are needed when editing a request target directly.
- `&` is especially important because unencoded ampersands may split query parameters.
- Double or incorrect encoding can turn intended syntax into literal text.

## Learning struggle and corrections

- I first tested the `/login` endpoint because the goal involved an administrator password. The lab's vulnerable feature was not the login request.
- I then tested `/my-account?id=...`, but this route returned redirects and was not the actual user lookup.
- I found the real `/user/lookup` request in Burp HTTP history.
- I initially tried to place a regex string into the expression without applying it to `this.password`.
- I tried creating a separate `password` query parameter, but the vulnerable endpoint used only the `user` expression.
- I had a request with incorrect and repeated encoding such as `%u...` sequences.
- I learned that `this.password[0]` indexes the first character of a JavaScript string; the password does not need to be an array.
- I learned why Cluster Bomb was appropriate for every combination of password position and letter, while Pitchfork would pair payloads rather than test the full combination set.

## Current understanding

I should be able to explain:

- what input I control,
- which endpoint actually processes it,
- whether the input is a value, object, operator, or syntax,
- what parser/query engine processes it,
- what evidence proves the query changed,
- how an oracle can leak data without displaying it directly,
- what the root cause is,
- how a developer should fix it,
- what regression tests should prevent recurrence.

I do not need to memorise every operator or payload. I need to understand data structures, query construction, evidence, impact, and remediation.
