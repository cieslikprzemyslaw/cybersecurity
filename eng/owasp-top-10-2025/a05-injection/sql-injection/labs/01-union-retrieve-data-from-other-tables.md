# Lab: UNION-Based SQL Injection - Retrieving Data from Other Tables

## Source

PortSwigger Web Security Academy: **SQL injection UNION attack, retrieving data from other tables**

## Goal

Exploit a UNION-based SQL Injection vulnerability to retrieve data from another table and log in as the administrator.

## Vulnerable feature

The vulnerable feature was the product category filter.

Controlled input:

```text
/filter?category=Pets
```

The `category` parameter appeared to be used in a backend SQL query.

## Interpreter involved

The relevant interpreter was the database / SQL engine.

## Testing flow

### 1. Normal request

A normal category value returned `200 OK` and displayed products.

### 2. Quote test

Adding a single quote caused a `500 Internal Server Error`.

This suggested that the quote broke the SQL syntax.

### 3. SQL comment test

Adding a SQL comment returned the page to normal behaviour.

This suggested that the quote closed the SQL string and the comment removed the problematic rest of the original query.

### 4. Column count

Testing with one `NULL` caused an error.

Testing with two `NULL` values returned a valid response.

Conclusion:

```text
The original query likely returned 2 columns.
```

### 5. Text-compatible columns

Testing with text values such as `abc` and `def` returned both values in the HTML response.

This confirmed:

- the UNION result was visible,
- both columns could handle text,
- the injected SELECT result was rendered in the page.

### 6. Data extraction

The next step was to retrieve usernames and passwords from the `users` table.

The response showed users including:

```text
administrator
```

The administrator password was then used to log in and solve the lab.

## Evidence summary

```text
Normal category -> 200 OK
Single quote -> 500 error
Quote + comment -> 200 OK
UNION SELECT NULL -> error
UNION SELECT NULL,NULL -> 200 OK
UNION SELECT 'abc','def' -> values visible in HTML
UNION SELECT username,password FROM users -> credentials visible in HTML
```

## Real impact

The issue allowed sensitive data disclosure from another table, including usernames and passwords. This led to administrator account compromise.

## Mistakes and learning process

- I initially needed a reminder to use a SQL comment after breaking the query with a quote.
- I learned that `NULL,NULL` helps identify column count, but does not prove data extraction by itself.
- The `abc/def` test was important because it proved that injected text values were visible in the response.
- This lab showed the difference between controlled evidence gathering and random payload guessing.

## Developer remediation

- Use prepared statements / parameterized queries.
- Do not concatenate category values into SQL query strings.
- Use an allowlist for known category values.
- Avoid verbose database errors.
- Use least-privilege database accounts.
- Add regression tests for SQLi payloads.

## Regression test ideas

- A single quote in `category` should not cause a 500 error.
- SQL comments should not affect query behaviour.
- `UNION SELECT NULL,NULL` should not return additional rows.
- Injected text values should not appear in the response.
- Attempts to select from `users` should fail safely.
