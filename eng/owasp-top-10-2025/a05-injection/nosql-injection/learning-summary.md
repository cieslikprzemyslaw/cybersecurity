# NoSQL Injection Learning Summary

## Status

**PASS**

I completed the NoSQL Injection theory section, TryHackMe exercises, and two PortSwigger labs.

## What I understand now

- MongoDB stores documents in collections.
- MongoDB filters are structured objects rather than SQL strings.
- Multiple field conditions commonly behave like an implicit `AND`.
- Operators such as `$ne`, `$nin`, and `$regex` can change query meaning.
- Operator Injection often happens when a value expected to be a string becomes a nested query object.
- Syntax Injection can happen when input is concatenated into custom JavaScript-style query logic.
- Syntax Injection is generally less common than Operator Injection.
- A single quote may provide evidence, but true/false confirmation is stronger.
- A boolean oracle can leak a secret without displaying it.
- Background API requests may be more important than the visible page URL.
- JavaScript strings use zero-based character indexing.
- Intruder should automate a working manual test, not replace the reasoning stage.

## Theory lesson

The most important distinction was:

```text
Operator Injection:
input becomes a query object/operator

Syntax Injection:
input breaks out of an expression and adds syntax
```

## PortSwigger lab 1

The category filter:

```text
/filter?category=Gifts
```

returned a normal product set.

Adding a quote caused a MongoDB JavaScript interpreter error.

An always-true JavaScript condition returned products outside the selected category.

This confirmed NoSQL Syntax Injection.

## PortSwigger lab 2

The first challenge was finding the real attack surface.

I initially tested:

```text
/login
```

and then:

```text
/my-account?id=wiener
```

The vulnerable request was a background API lookup:

```text
/user/lookup?user=wiener
```

A quote changed the response to an application error.

An always-true condition returned the administrator record instead of `wiener`.

A length condition confirmed an eight-character password. Character conditions then tested indexes `0-7`. Burp Intruder used Cluster Bomb to combine every index with every lowercase letter.

The recovered lab password was used to complete the exercise. The password itself is intentionally not recorded because the method and evidence are the useful learning outcomes.

## Mistakes that improved my understanding

### Wrong endpoint

I assumed the login endpoint was vulnerable because the goal involved a password.

Correction:

```text
The goal does not identify the vulnerable feature.
HTTP history and application behaviour identify it.
```

### Visible URL vs API request

I focused on `/my-account`, but the page loaded data through `/user/lookup`.

Correction:

```text
Inspect background requests, not only navigation requests.
```

### Regex as a literal string

I attempted to add a pattern without applying it to the password field.

Correction:

```text
A regex pattern does not perform a test by itself.
It must be used through an operator or function.
```

### Separate password parameter

I tried adding `&password=...`.

Correction:

```text
Adding a new parameter does not make the vulnerable query use it.
The condition had to be injected into the actual expression.
```

### Encoding

The Intruder request contained incorrect `%u...` and repeated encoding.

Correction:

```text
Use normal URL encoding once and confirm the server receives the intended expression.
```

### String indexing

I initially described `this.password[0]` as array indexing.

Correction:

```text
JavaScript strings can also be indexed.
Index 0 is the first character.
```

### Cluster Bomb vs Pitchfork

I had not used Pitchfork before.

I learned:

```text
Cluster Bomb -> every combination
Pitchfork     -> paired values
```

Cluster Bomb matched the requirement of every index combined with every candidate character.

## Interview-style takeaway

NoSQL Injection is not only about inserting special characters. It often depends on how the backend parses request data and whether an expected primitive value can become an object, operator, or executable expression.

My testing approach is:

1. Find the real processing endpoint.
2. Capture a baseline.
3. Identify the controlled input and data type.
4. Test one controlled syntax or structure change.
5. Compare true and false behaviour.
6. Confirm impact.
7. Explain root cause.
8. Recommend type enforcement, schema validation, server-controlled query construction, secure password verification, and regression tests.
