# UNION-Based SQL Injection

## What it is

UNION-based SQL Injection uses the SQL `UNION` operator to combine the result of the original query with the result of an injected query.

The goal is usually to make the application display data from another table.

## Key rule

`UNION SELECT` must return the same number of columns as the original query.

If the original query returns 2 columns, the injected query must also return 2 columns:

```sql
UNION SELECT NULL,NULL
```

If the number of columns is wrong, the database will usually return an error.

## Why NULL is useful

`NULL` is useful because it is compatible with many column types.

It helps identify the number of columns without needing to know the exact data types yet.

Important distinction:

```text
NULL,NULL confirms column count.
It does not yet prove useful data extraction.
```

## Text-compatible column testing

After finding the column count, test whether text values can be displayed:

```sql
UNION SELECT 'abc','def'
```

If `abc` and `def` appear in the response, it proves that the injected result is visible in the HTML and that the columns can hold text.

## Lab evidence flow

In the PortSwigger lab, the evidence flow was:

```text
normal category value -> 200 OK
single quote -> 500 error
single quote + SQL comment -> 200 OK
UNION SELECT NULL -> 500 error
UNION SELECT NULL,NULL -> 200 OK
UNION SELECT 'abc','def' -> abc and def appeared in HTML
UNION SELECT username,password FROM users -> usernames and passwords appeared in HTML
```

## Real impact

UNION-based SQLi can allow attackers to retrieve sensitive data from other tables, such as usernames and passwords.

In the lab, this led to administrator account compromise.

## Developer fix

Use prepared statements / parameterized queries. For category filters, also use allowlisted category values where appropriate.
