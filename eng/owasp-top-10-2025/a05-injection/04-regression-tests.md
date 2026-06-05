# A05: Injection - Regression Test Ideas

Regression tests should prove that previously dangerous input is now treated as data, rejected safely, or prevented from reaching executable query syntax.

## SQL Injection regression tests

- A single quote in a category or search value should not cause a 500 error.
- SQL comments such as `--` should not change query behaviour.
- Boolean payloads such as `' AND '1'='1` and `' AND '1'='2` should not produce meaningful response differences.
- `UNION SELECT NULL,NULL` should not return additional rows or data.
- `UNION SELECT 'abc','def'` should not render attacker-controlled SQL result data.
- Cookie values such as `TrackingId` should not be usable to alter SQL query logic.
- SQLi payloads should be handled safely as ordinary strings.

## NoSQL Injection regression tests

### Primitive type enforcement

- `username` and `password` accept strings only.
- Arrays, objects, and nested structures are rejected with a controlled `400` response.
- JSON such as `{"username":{"$ne":null}}` is rejected.
- Form data using bracket notation to create nested objects is rejected where a string is expected.
- Unknown fields are rejected or removed by schema validation.

### Query operator control

- Client input cannot introduce `$ne`, `$nin`, `$gt`, `$regex`, `$where`, or other query operators.
- A value containing `$` remains data or is rejected; it does not become a query key.
- A user cannot control field names, projections, sort expressions, or raw filters unless explicitly allowlisted.
- Operator-like input does not bypass authentication or return the first database document.

### Syntax Injection

- A quote in a lookup or category value does not cause a MongoDB or JavaScript interpreter error.
- JavaScript-style boolean expressions do not change result sets.
- Always-true expressions do not return unreleased products, unrelated records, or another user's data.
- No user-controlled input is concatenated into `$where` or custom JavaScript.

### Boolean-oracle resistance

- True and false payloads do not produce a meaningful secret-dependent response difference.
- User lookup responses do not expose whether a condition about a password or secret is correct.
- Error bodies, response lengths, and status codes are normalised where appropriate.
- Repeated probing triggers rate limiting, alerting, or temporary controls.

### Authentication design

- The application retrieves a user only by an exact, validated username or email.
- Password comparison uses a secure password-hash verification function.
- Passwords are not stored or queried as plaintext.
- Failed authentication does not return another user record.
- Authentication success cannot be triggered only because a query returned at least one document.

## Application behaviour expectations

- Invalid input returns a controlled response, not a raw database error.
- Unknown categories or users return no result or a safe validation response.
- The application does not expose raw SQL, MongoDB, JavaScript, driver, or stack-trace details.
- Background API endpoints enforce the same validation and authorisation as visible pages.

## Code-level checks

- SQL queries use prepared statements or parameterized queries.
- NoSQL filters are built from server-selected fields and operators.
- Request data is converted into a validated internal model before query construction.
- ORM/ODM usage avoids unsafe raw query construction.
- Database users have only the permissions required by the application.

## Security finding regression checklist

For each injection fix, confirm:

- the vulnerable parameter no longer changes interpreter behaviour,
- dangerous input is treated as data or rejected,
- response differences no longer expose secret-dependent truth values,
- sensitive records cannot be retrieved,
- administrator account compromise is no longer possible,
- automated tests prevent the issue from returning.
