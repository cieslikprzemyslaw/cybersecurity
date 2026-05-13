# SQL Injection - Learning Notes

> Status: Learning notes  
> Scope: Legal labs and intentionally vulnerable applications only  
> Focus: understanding SQL Injection from a developer and AppSec perspective

## TL;DR

SQL Injection happens when user-controlled input is inserted into a SQL query in an unsafe way, allowing the user to change the meaning of the query.

The useful lesson is not only that a payload works. The useful lesson is understanding:

- where the input enters the query,
- what SQL context the input is in,
- how the query logic changes,
- what impact this could have,
- and how the issue should be fixed properly.

The main fix is not filtering quotes. The main fix is using parameterized queries / prepared statements.

## What I Practised

So far, I practised SQL Injection in three areas:

- login bypass,
- manipulating a `WHERE` clause to retrieve hidden data,
- using `UNION SELECT` to determine the number of columns returned by a query.

The goal was not to memorise payloads, but to understand how user input can affect backend SQL logic.

## Mental Model

When testing for SQL Injection in a lab, I try to ask:

- What input can I control?
- Is the input used in a SQL query?
- Is the input inside a string, number, filter, or another SQL context?
- Can I break out of the original context?
- Can I change the query logic?
- Does the response show an error, redirect, different data, or no visible change?
- Is another mechanism blocking the request, such as CSRF or session validation?

This helped me avoid treating every failed request as a failed SQLi attempt. Sometimes the SQLi idea is correct, but the request fails for another reason.

## Key Observations

### 1. A redirect can be a success signal

In the login bypass lab, a `302 Found` response with a redirect to the account page was an important clue that the login had worked.

### 2. CSRF/session errors are not the same as SQLi errors

An invalid CSRF token did not mean the SQL Injection payload was wrong. It meant the request state was wrong because the CSRF token and session cookie did not match.

### 3. Reflected text is not always executed SQL logic

When SQL-like text appeared in a page title, it did not automatically mean the SQL condition had executed. The input still needed to break out of the original SQL string context.

### 4. `UNION SELECT` needs structure compatibility

`UNION SELECT` requires the same number of columns as the original query and compatible data types. Using `NULL` is useful when testing column count because it is compatible with many SQL column types.

## Common Mistakes I Noticed

- Testing the wrong parameter first.
- Adding SQL keywords without escaping the original context.
- Treating every error as the same kind of failure.
- Looking only at the visible page instead of checking status codes, headers, redirects, cookies, and HTML differences.
- Forgetting that `UNION SELECT` needs both matching column count and compatible data types.

## Developer Remediation

SQL Injection should be fixed by separating SQL code from user data.

The main remediation is to use:

- parameterized queries,
- prepared statements,
- safe ORM query methods where appropriate.

Additional protections include:

- input validation based on expected type and format,
- least privilege database accounts,
- generic error messages,
- logging and monitoring of suspicious behaviour,
- security-focused regression tests,
- code review for unsafe query construction.

Escaping or filtering quotes is not the main fix. It can be bypassed or applied inconsistently. The safer approach is to avoid building SQL queries through string concatenation.

## Developer-Focused Review Questions

When reviewing code, I would ask:

- Is user input being concatenated into a SQL query?
- Are parameterized queries used?
- Is the input expected to be a string, number, enum, or identifier?
- Is the query logic affected by user-controlled values?
- Can the user bypass filters by changing request parameters?
- Are authorization checks separate from filtering logic?
- Are database errors exposed to the user?
- Is there a regression test for unexpected or malicious input?

## Still Reviewing

Areas I still want to strengthen:

- explaining the difference between error-based, boolean-based, time-based, and UNION-based SQLi,
- identifying SQL context faster: string, number, `WHERE`, `ORDER BY`, `LIKE`, etc.,
- explaining prepared statements in simple developer-friendly language,
- deciding whether a failure is caused by SQLi syntax, CSRF/session state, validation, or data type mismatch.

## Main Takeaway

SQL Injection is not just about payloads. It is about understanding how user-controlled input changes backend query logic.

From a frontend perspective, this reinforces an important point:

> The frontend is not the security boundary.

Even if the UI only shows allowed filters or actions, the backend must safely handle modified requests.
