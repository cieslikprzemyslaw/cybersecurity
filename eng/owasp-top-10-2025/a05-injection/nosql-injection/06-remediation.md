# NoSQL Injection Remediation

NoSQL Injection should be fixed by controlling input types and query structure, not by adding a small blacklist of characters.

## 1. Enforce strict request schemas

Credential and lookup fields should have exact expected types.

Example rules:

```text
username:
- required
- string
- maximum expected length
- no arrays
- no objects

password:
- required
- string
- no arrays
- no objects
```

Reject invalid structures before query construction.

Dangerous shape:

```json
{
  "username": {
    "$ne": null
  }
}
```

Expected safe shape:

```json
{
  "username": "alice"
}
```

## 2. Build query structure on the server

The client should supply values, not database filters.

Avoid patterns conceptually similar to:

```javascript
collection.find(req.body)
```

Prefer selecting the fields and operators in trusted server code:

```javascript
const filter = {
  username: validatedUsername
};
```

This still requires type validation. A server-created object is not automatically safe if the inserted value can itself be a nested object.

## 3. Reject operator-shaped input

Where an endpoint expects simple values:

- reject objects,
- reject arrays,
- reject nested structures,
- reject unexpected fields,
- do not merge request objects into query filters,
- do not allow client-selected database operators.

A blacklist of `$ne` alone is weak because there are many operators and parser variations.

## 4. Avoid custom JavaScript query construction

Do not concatenate input into:

- `$where`,
- JavaScript expressions,
- dynamically evaluated functions,
- raw query strings interpreted by the database.

Risky pattern:

```javascript
"$where": "this.username == '" + username + "'"
```

Prefer built-in structured queries with validated primitive values.

## 5. Design authentication correctly

Do not query the database using both username and plaintext password.

Safer flow:

1. Validate username/email and password as strings.
2. Query the user by an exact validated username/email.
3. If the user exists, verify the submitted password against a secure password hash.
4. Create a session only after successful hash verification.
5. Return a generic failure message otherwise.

Conceptual Node.js-style example:

```javascript
if (typeof username !== "string" || typeof password !== "string") {
  return response.status(400).json({ message: "Invalid request" });
}

const user = await users.findOne({ username });

if (!user) {
  return response.status(401).json({ message: "Invalid credentials" });
}

const passwordValid = await verifyPasswordHash(
  user.passwordHash,
  password
);

if (!passwordValid) {
  return response.status(401).json({ message: "Invalid credentials" });
}
```

The exact password library depends on the application stack.

## 6. Use allowlists for dynamic features

Some APIs legitimately support filtering or sorting.

When a MongoDB-backed API exposes any filter feature, compare the allowed operations with the official MongoDB [Query Predicates](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/) reference and allow only the small subset the feature genuinely needs.

The server should allowlist:

- permitted filter fields,
- permitted comparison operations,
- sort directions,
- projection fields,
- maximum page sizes.

Translate safe client options into server-created database filters. Do not accept raw database query objects.

## 7. Control error handling

Production responses should not expose:

- MongoDB driver errors,
- JavaScript interpreter errors,
- stack traces,
- internal file paths,
- database addresses,
- query fragments.

Log detailed errors securely on the server and return a controlled client response.

## 8. Reduce oracle quality

Do not rely on response normalisation as the primary fix, but use it as defence in depth.

- generic authentication errors,
- controlled lookup responses,
- stable response shapes where appropriate,
- rate limiting,
- account and IP monitoring,
- anomaly detection,
- alerts for repeated character-position probing.

## 9. Apply least privilege

The application database account should have only required access.

Limit:

- collections,
- read/write operations,
- administrative functions,
- scripting features where not required,
- outbound connectivity where relevant.

## 10. Add regression tests

Tests must include both textual payloads and structured input.

Examples:

- quote characters remain data,
- nested objects are rejected,
- `$ne` objects cannot bypass login,
- `$regex` cannot create a password oracle,
- always-true expressions cannot broaden results,
- verbose database errors are not returned,
- background API endpoints apply the same controls as visible forms.

## Developer takeaway

The secure boundary is:

```text
untrusted request
-> schema validation and type enforcement
-> trusted internal values
-> server-controlled query structure
-> database
```

The application should never allow the client to decide where query syntax begins.
