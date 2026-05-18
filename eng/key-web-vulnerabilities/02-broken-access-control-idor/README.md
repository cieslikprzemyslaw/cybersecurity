# Broken Access Control and IDOR

This is the second topic in the Key Web Vulnerabilities series.

## Start Here

1. [Overview](overview.md)
2. [Module Summary](summary.md)
3. [Practical Cheat Sheet](cheat-sheet.md)
4. [Labs](labs/README.md)

## Topic Focus

This module covers:

- authentication versus authorization,
- horizontal and vertical access control,
- insecure direct object references,
- user-controlled object identifiers,
- server-side permission checks,
- developer-focused remediation.

## Completed Labs

| Platform | Lab / Room | Notes |
|---|---|---|
| TryHackMe | Lab 01 — Broken Access Control | [Lab summary](labs/lab-01-tryhackme-broken-access-control.md) |
| TryHackMe | Lab 02 — IDOR | [Lab summary](labs/lab-02-tryhackme-idor.md) |
| PortSwigger | Lab 03 — User ID controlled by request parameter | [Lab summary](labs/lab-03-portswigger-user-id-controlled-by-request-parameter.md) |
| PortSwigger | Lab 04 — Insecure direct object references | [Lab summary](labs/lab-04-portswigger-insecure-direct-object-references.md) |

## Main Takeaway

```text
Authenticated does not mean authorized.
```

The backend must verify whether the current user is allowed to access the exact object or perform the exact action requested.
