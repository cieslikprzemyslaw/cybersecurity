# Lab 03 - UNION SELECT Column Count

> Topic: Determining how many columns the original query returns  
> Focus: `UNION SELECT` structure compatibility

## Goal

Find the number of columns returned by the original product listing query so that a `UNION SELECT` query can be accepted.

## Input I Controlled

The useful input was again the `category` query parameter:

```http
GET /filter?category=Gifts
```

I initially tested a product ID parameter, but the application returned `Invalid product ID`, which suggested that this was not the right injection point for this lab.

## What Happened

I started testing `UNION SELECT` with different numbers of selected values.

Using numeric values caused internal server errors. This showed that the issue might not only be the number of columns, but also data type compatibility.

Using `NULL` values was more useful because `NULL` is compatible with many SQL column types.

When three `NULL` values were used, the response changed to `HTTP/2 200 OK`. The page did not visibly change much, but the HTML contained an extra empty table row. That suggested the `UNION SELECT` was accepted and added a row to the result set.

## What I Learned

`UNION SELECT` requires:

- the same number of columns as the original query,
- compatible data types,
- and a response location where injected values may be visible.

A normal-looking response can still contain evidence that the query changed. Sometimes the important difference is in the HTML, not in the visible browser page.

## Impact

Finding the column count is an early step in UNION-based SQL Injection. It can allow an attacker to prepare a query that later extracts visible data from the database.

## Remediation Summary

See `../sql-injection-overview.md` for the general remediation section.

Specific to listing pages:

- use parameterized queries for filters,
- avoid building dynamic SQL directly from query parameters,
- hide detailed database errors,
- add regression tests for unexpected category values.

## Main Takeaway

The correct column count was confirmed when the server stopped returning an internal error and accepted the `UNION SELECT`. `NULL` helped avoid unnecessary data type problems while testing the query structure.
