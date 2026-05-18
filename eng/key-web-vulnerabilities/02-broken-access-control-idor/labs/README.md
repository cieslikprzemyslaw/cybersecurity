# Broken Access Control & IDOR — Labs

> Short summaries of legal training labs completed for this topic.  
> These are practice notes, not full walkthroughs or answer dumps.

---

## Labs Covered

| Platform | Lab / Room | Focus | Notes |
|---|---|---|---|
| TryHackMe | Lab 01 — Broken Access Control | AuthN vs AuthZ, access control categories, server-side checks | [Summary](lab-01-tryhackme-broken-access-control.md) |
| TryHackMe | Lab 02 — IDOR | Object references, changing IDs, checking backend behaviour | [Summary](lab-02-tryhackme-idor.md) |
| PortSwigger | Lab 03 — User ID controlled by request parameter | Horizontal privilege escalation through a user-controlled ID | [Summary](lab-03-portswigger-user-id-controlled-by-request-parameter.md) |
| PortSwigger | Lab 04 — Insecure direct object references | IDOR through chat transcript/file references | [Summary](lab-04-portswigger-insecure-direct-object-references.md) |

---

## Practice Focus

- Finding user-controlled object references.
- Modifying IDs, filenames, and object references.
- Comparing server responses.
- Thinking about ownership checks.
- Understanding that the frontend is not a security boundary.
- Writing developer-friendly remediation notes.

---

## Main Pattern

```text
User-controlled object reference + missing server-side authorization = Broken Access Control
```

## Cross-Lab Lessons

- IDOR is a type of Broken Access Control.
- The risky value is not always called `id`.
- Object references can be query parameters, paths, request body values, filenames, download links, cookies, headers, or encoded values.
- A frontend check can improve UX, but it cannot enforce authorization.
- The most reliable test pattern is to compare access between two different accounts.
- Secure implementations should check ownership or permissions on the server before returning or changing data.

## Useful Follow-Up

- [Module overview](../overview.md)
- [Module summary](../summary.md)
- [Practical cheat sheet](../cheat-sheet.md)

---

## Safe Testing Reminder

These notes are based on legal labs and authorised training platforms only.

Do not test real applications without permission, clear scope, and safe harbour.
