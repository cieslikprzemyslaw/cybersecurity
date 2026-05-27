# A02:2025 - Security Misconfiguration Learning Notes

These notes preserve the longer learning reflections from the A02 labs. The shorter lab evidence stays in [02-labs-or-practice.md](02-labs-or-practice.md).

## What I Learned

### 1. Security Misconfiguration Often Appears as Information Disclosure

A02 is not always about a complex exploit. It is often about something that should not be visible but is visible because of unsafe configuration.

Examples from these labs:

- verbose error output,
- traceback disclosure,
- public debug endpoint,
- `phpinfo()` exposure,
- environment/configuration leak,
- exposed application secret.

### 2. Invalid Input Is Useful for Testing Error Handling

When an application says:

```text
User ID must be numeric
```

it is useful to test what happens with non-numeric input.

The goal is not always SQL Injection or IDOR. Sometimes the important result is how the application fails.

Useful test values:

```text
admin
abc
test
-1
0
1.5
999999999999999999999
'
%27
```

### 3. HTML Comments Can Leak Useful Recon Information

HTML comments may reveal:

- hidden endpoints,
- old functionality,
- debug tools,
- internal notes,
- temporary links,
- development-only routes.

A comment does not secure anything. If it is sent to the browser, it is visible.

### 4. Debug Pages Are Dangerous in Public Environments

Debug pages such as `phpinfo()` can expose a large amount of sensitive technical information.

Even if some settings look safe, such as `display_errors=Off`, the page itself may still disclose environment details, paths, modules, and secrets.

### 5. Secrets Must Not Be Stored in Error Messages or Exposed Debug Output

If a secret is exposed, the fix must include:

- removing the exposure,
- rotating the secret,
- checking where else it was used,
- reviewing logs and deployment artifacts,
- adding secret scanning.

## What Confused Me or Needed Clarification

### 1. `400` vs `404` vs `403`

For invalid input to an existing API endpoint, `400 Bad Request` is usually appropriate.

Example:

```http
GET /api/user/admin
```

The endpoint exists, but the input is invalid.

For a debug page that should not exist publicly, `404 Not Found` is usually better.

Example:

```http
GET /cgi-bin/phpinfo.php
```

If the diagnostic endpoint intentionally exists but only authorised users should access it, then `403 Forbidden` is appropriate for authenticated users without permission.

### 2. The Comment Was Not the Vulnerability by Itself

The HTML comment was a discovery clue.

The real vulnerability was the public debug endpoint and the exposed configuration/secret.

### 3. Removing the Comment Is Not Enough

Removing the HTML comment reduces discoverability, but it does not fix the issue if `/cgi-bin/phpinfo.php` still exists.

The endpoint must be removed, blocked, or strictly protected.

### 4. Changing the URL Is Not a Proper Fix

Changing `/cgi-bin/phpinfo.php` to another hidden URL is security by obscurity.

A better fix is to remove the debug file from production or restrict it properly.

## Real-World Review Angle

In a real application, I would look for this A02 pattern in:

- HTML comments,
- JavaScript bundles,
- source maps,
- robots.txt,
- sitemap.xml,
- old test routes,
- admin/debug links,
- `/debug`,
- `/config`,
- `/status`,
- `/health`,
- `/server-status`,
- `/actuator`,
- `/phpinfo.php`,
- `/cgi-bin/phpinfo.php`,
- `.env`,
- `.bak`,
- `.old`,
- log files,
- verbose API errors,
- stack traces,
- response headers,
- deployment artifacts.

I would also review whether debug tools or diagnostic pages are deployed to production.

## Summary

The two practical A02 patterns I completed were:

1. invalid input causing verbose debug/traceback disclosure,
2. public debug endpoint exposing `phpinfo()` and a sensitive application secret.

The main lesson:

> Production systems should not expose debug information, stack traces, diagnostic pages, secrets, environment details, or internal configuration to users.
