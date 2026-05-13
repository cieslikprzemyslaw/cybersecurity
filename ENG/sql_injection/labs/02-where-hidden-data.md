# Lab 02 - SQL Injection in a WHERE Clause

> Topic: Changing filtering logic to retrieve hidden data  
> Focus: understanding SQL string context and application filters

## Goal

Modify a product category filter so that the application displays products that are normally hidden by a `released = 1` condition.

## Input I Controlled

The controlled input was the `category` query parameter:

```http
GET /filter?category=Accessories
```

The lab description indicated that the backend query was similar to:

```sql
SELECT * FROM products
WHERE category = 'Gifts'
AND released = 1
```

## What Happened

The homepage request did not contain an obvious useful parameter, but selecting a category created a request with a `category` parameter.

I first tried adding SQL-like text without breaking out of the string context. The text appeared on the page as the category title, but it did not mean the SQL condition had executed.

The working idea was to first leave the original string context, then change the `WHERE` logic, and then prevent the original `released = 1` condition from still applying.

## What I Learned

Reflected text is not the same as executed SQL logic.

If the input is still inside a quoted string, then SQL keywords can simply become part of the string value. The SQL logic only changes when the input breaks out of the original context.

## Impact

A vulnerable filter can expose records that should not be visible, such as unpublished products, hidden content, private records, or internal data.

## Remediation Summary

See `../sql-injection-overview.md` for the general remediation section.

Specific to filters:

- use parameterized queries,
- validate category values against an allowlist or known enum,
- do not rely on frontend filters to protect hidden records,
- keep authorization and visibility rules enforced server-side.

## Main Takeaway

SQL Injection in a `WHERE` clause can change how application filters work. The key lesson was that SQL-like text must escape the original string context before it can affect query logic.
