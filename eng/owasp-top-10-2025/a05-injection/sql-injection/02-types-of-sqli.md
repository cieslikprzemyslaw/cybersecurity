# Types of SQL Injection

SQL Injection can be grouped into several major types. Understanding the type matters because each one has a different evidence pattern.

## Main SQLi types

```text
SQL Injection
├── In-band SQLi
│   ├── Error-based SQLi
│   └── Union-based SQLi
├── Inferential / Blind SQLi
│   ├── Boolean-based Blind SQLi
│   └── Time-based Blind SQLi
└── Out-of-band SQLi
```

## In-band SQL Injection

In-band SQLi means the attacker uses the same communication channel for both the attack and the result.

### Error-based SQLi

The attacker causes database errors and uses the error messages to learn about the database structure, query behaviour, or database type.

Evidence:

```text
Database error messages appear in the response.
```

### Union-based SQLi

The attacker uses `UNION SELECT` to append results from another SELECT query into the normal application response.

Evidence:

```text
Data from another table appears in the web response.
```

## Inferential / Blind SQL Injection

Blind SQLi does not return database data directly. Instead, the attacker infers information from application behaviour.

### Boolean-based blind SQLi

The attacker sends true and false SQL conditions and observes whether the response changes.

Evidence:

```text
TRUE condition -> one response behaviour
FALSE condition -> different response behaviour
```

### Time-based blind SQLi

The attacker sends a condition that causes a delay if the condition is true.

Evidence:

```text
The response is delayed only when the injected condition is true.
```

## Out-of-band SQL Injection

Out-of-band SQLi uses a separate channel for the result or evidence, such as DNS, HTTP, or SMB.

Evidence:

```text
External callback, external request, or file written to an attacker-controlled location.
```

## AppSec note

The type of SQLi changes the evidence, but the root cause is the same: unsafe construction of SQL queries using untrusted input.
