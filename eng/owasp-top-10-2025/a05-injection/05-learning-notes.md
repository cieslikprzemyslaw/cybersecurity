# A05: Injection - Learning Notes

## What I learned so far

SQL Injection helped me understand the wider A05 Injection model:

```text
input -> interpreter -> changed behaviour
```

The key lesson is that the dangerous part is not the payload by itself. The dangerous part is when the application allows user input to become part of SQL syntax.

## Important SQL Injection observations

- A single quote can break a SQL query when input is inserted into a string context.
- Adding a SQL comment can repair the query by commenting out the remaining SQL.
- `UNION SELECT` requires the same number of columns as the original query.
- `NULL` is useful for testing column count because it is compatible with many column types.
- `NULL,NULL` confirms structure, but it does not prove data extraction by itself.
- Testing with `'abc','def'` proved that text columns were visible in the HTML response.
- Blind SQLi required a different mindset because the data was not shown directly.

## Encoding lesson

In one THM task, an encoded payload appeared in the generated SQL query as literal encoded text, such as `%20`, `%27`, or `%7C`. That meant the payload was not decoded at the level I expected and was treated as text rather than SQL syntax.

Important lesson:

```text
Raw payload in UI if the frontend encodes input.
Encoded payload in direct URL or Burp Repeater.
```

## Blind SQLi lesson

Blind SQL Injection is about building a reliable true/false oracle.

In the PortSwigger lab:

```text
Welcome back = TRUE
No Welcome back = FALSE
```

This allowed me to confirm the users table, confirm the administrator user, determine the password length, and extract the password character by character.

## Mistakes and corrections

- I initially forgot that SQL comments are important after breaking a query with a quote.
- I confused exact interpreter terminology at first. In SQL Injection, the relevant interpreter is the database / SQL engine, not the PHP connection object.

## Current understanding

For my current AppSec level, I should be able to explain:

- what input I control,
- where it is processed,
- what evidence proves SQLi,
- what the real impact is,
- how a developer should fix it.

I do not need to memorise every advanced SQLi payload for every database, but I should understand the mechanisms, impact, and remediation.

The detailed SQLi lab comparison is kept in [sql-injection/learning-summary.md](sql-injection/learning-summary.md), so this file stays focused on the wider A05 learning model.
