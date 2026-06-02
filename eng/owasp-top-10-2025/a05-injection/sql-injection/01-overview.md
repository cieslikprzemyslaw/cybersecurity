# SQL Injection - Overview

## Definition

SQL Injection is a vulnerability where user-controlled input is included in a SQL query and changes the logic executed by the database.

A vulnerable pattern looks like this:

```php
$book_name = $_GET['book_name'];
$sql = "SELECT * FROM books WHERE book_name = '$book_name'";
```

The problem is that `$book_name` is placed directly inside the SQL string. If the user provides SQL syntax, the database may treat part of the input as executable query logic.

## Why it matters

SQL Injection can lead to:

- authentication bypass,
- sensitive data disclosure,
- extraction of usernames and passwords,
- modification of data,
- deletion of data,
- possible privilege escalation depending on database and configuration.

## Interpreter involved

The relevant interpreter is the **database / SQL engine**, for example MySQL, MariaDB, PostgreSQL, MSSQL, Oracle, or another SQL database.

PHP, Node.js, Java, or other backend code may build and send the query, but the SQL engine interprets the final SQL statement.

## Root cause

The root cause is usually:

```text
Combining untrusted input with SQL query text.
```

Validation can help, but the real protection comes from query parameterization.

## Main remediation

Use prepared statements / parameterized queries.

Conceptually:

```text
SQL structure: SELECT * FROM books WHERE book_name = ?
User input:    passed separately as a value
```

This prevents user input from becoming SQL syntax.
