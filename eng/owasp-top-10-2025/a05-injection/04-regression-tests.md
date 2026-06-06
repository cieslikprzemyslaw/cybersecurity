# A05: Injection - Regression Test Ideas

Regression tests should prove that previously dangerous input is now treated as data, rejected safely, or prevented from reaching executable query or command syntax.

## SQL Injection regression tests

- A single quote in a category or search value should not cause a `500` error.
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

## OS Command Injection regression tests

### Architectural checks

- The feature uses a language library, SDK, or service API instead of invoking an operating-system command where possible.
- No user-controlled value reaches `sh -c`, `bash -c`, `cmd /c`, PowerShell command strings, or an equivalent shell interpreter.
- The executable is selected by server-side code and is not client-controlled.
- Command and arguments are passed as separate values.
- The process API is configured not to use a shell.

### Input and argument handling

- Shell metacharacters are rejected by a strict allowlist or remain literal data without changing execution.
- Separators such as `;`, `&`, `&&`, `|`, and `||` do not launch extra commands.
- Newlines and command-substitution syntax do not alter process execution.
- Quotes do not break out of an argument context.
- Values beginning with `-` cannot become unexpected command options.
- An end-of-options delimiter such as `--` is used where supported and appropriate.
- Command names and required flags are hardcoded.
- Limited user choices are mapped from stable application identifiers to approved server-side values.

### Behaviour and side-effect checks

- A normal request and a test containing a timing marker have comparable response times within an agreed tolerance.
- Increasing a supplied delay value does not produce proportional response delays.
- Test input does not create unexpected files.
- Test input does not change file contents, application state, or child-process behaviour.
- Test input does not trigger outbound DNS, HTTP, or other network interactions.
- Command output cannot appear in the HTTP response, logs, files, or error messages through unsafe execution.
- Invalid input returns a controlled response rather than a raw process error or stack trace.

### Privilege and containment checks

- The application process runs as a dedicated low-privilege account.
- The process cannot write to the web root unless the feature explicitly requires it.
- The process cannot read unrelated secrets, configuration files, or user data.
- The process cannot execute unnecessary binaries.
- Child processes inherit only the environment variables and filesystem access they require.
- Process execution has timeouts, output-size limits, and resource limits.

### Code review and automated checks

- Static analysis flags unsafe uses of `exec`, `system`, `shell_exec`, `passthru`, `child_process.exec`, `shell=True`, `sh -c`, `cmd /c`, and similar patterns.
- Tests fail if a refactor reintroduces string concatenation around a process-execution sink.
- Tests verify that a safe argument array is used rather than a single command string.
- Tests cover expected values, boundary values, option-like values, whitespace, encoding, and metacharacters.
- Logging records denied or failed execution attempts without storing secrets.

## Application behaviour expectations

- Invalid input returns a controlled response, not a raw database, shell, process, or stack-trace error.
- Unknown categories or users return no result or a safe validation response.
- Background API endpoints enforce the same validation and authorisation as visible pages.
- A `500` response is investigated but is not used as the only proof that a security test succeeded or failed.

## Code-level checks

- SQL queries use prepared statements or parameterised queries.
- NoSQL filters are built from server-selected fields and operators.
- Request data is converted into a validated internal model before query construction.
- ORM/ODM usage avoids unsafe raw query construction.
- Process execution avoids the shell and uses a fixed executable with a separate argument list.
- Database and operating-system users have only the permissions required by the application.

## Security finding regression checklist

For each injection fix, confirm:

- the vulnerable parameter no longer changes interpreter behaviour,
- dangerous input is treated as data or rejected,
- response differences no longer expose secret-dependent truth values,
- timing and side effects no longer prove hidden execution,
- sensitive records or files cannot be retrieved,
- unexpected commands or child processes cannot be started,
- administrator or server compromise is no longer possible through the original path,
- automated tests prevent the issue from returning.

For the expanded OS Command Injection test set, see [os-command-injection/regression-tests.md](os-command-injection/regression-tests.md).
