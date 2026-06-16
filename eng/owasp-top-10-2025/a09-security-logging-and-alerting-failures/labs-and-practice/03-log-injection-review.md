# Log Injection Review

## Reviewed code

```ts
app.post("/login", async (request, response) => {
  const { username, password } = request.body;
  const user = await authenticate(username, password);

  if (!user) {
    logger.warn(
      `${new Date().toISOString()} LOGIN_FAILED username=${username} ip=${request.ip}`
    );

    return response.status(401).json({ message: "Invalid credentials" });
  }

  logger.info(
    `${new Date().toISOString()} LOGIN_SUCCESS username=${username} ip=${request.ip}`
  );

  return response.status(200).json({ message: "Logged in" });
});
```

## User-controlled values

- `username` comes directly from the request body.
- `password` also comes from the request body but is not inserted into the shown log line.
- `request.ip` depends on the Express and proxy configuration. Forwarded headers can be misleading when `trust proxy` is configured incorrectly.

## Construction problem

The event is composed as a plain string. A controlled value is inserted into the same text that defines event boundaries and field names.

```text
untrusted username
    -> interpolated line
    -> line parser / viewer / SIEM
```

## Potential impact

An attacker may be able to:

- forge a second-looking entry,
- corrupt the record structure,
- hide malicious activity,
- cause a false positive or false negative,
- assign activity to the wrong actor,
- mislead an investigator.

The exercise did not prove that existing stored records could be overwritten. That would normally require additional access or another vulnerability.

## Safer design

```ts
logger.warn("Authentication failed", {
  timestamp: new Date().toISOString(),
  eventName: "authn_login_fail",
  accountId,
  sourceIp: request.ip,
  requestId: request.id,
  result: "failure",
  reasonCode: "INVALID_CREDENTIALS",
});
```

Additional controls:

- structured serialisation,
- field-length limits,
- control-character handling,
- application-controlled event name, severity, result, and reason code,
- no password logging,
- safe downstream parsing and display.

## Regression test

1. Submit a username containing newline and control characters.
2. Cause authentication to fail.
3. Capture the emitted event.
4. Confirm exactly one event exists.
5. Confirm the event remains `authn_login_fail` with the expected severity and result.
6. Confirm the username is contained safely within one field.
7. Confirm no password or additional forged record exists.

## Main takeaway

> Untrusted data may be a field value, but it must not control the structure or meaning of the security event.
