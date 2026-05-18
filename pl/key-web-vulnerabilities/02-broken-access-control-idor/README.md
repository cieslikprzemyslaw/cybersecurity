# Broken Access Control i IDOR

To drugi temat w serii Key Web Vulnerabilities.

## Start

1. [Overview](overview.md)
2. [Podsumowanie modułu](summary.md)
3. [Praktyczna checklista](cheat-sheet.md)
4. [Laby](labs/README.md)

## Zakres tematu

Ten moduł obejmuje:

- różnicę między authentication i authorization,
- poziomą i pionową kontrolę dostępu,
- insecure direct object references,
- identyfikatory obiektów kontrolowane przez użytkownika,
- sprawdzanie uprawnień po stronie serwera,
- remediation z perspektywy developera.

## Ukończone laby

| Platforma | Lab / Room | Notatki |
|---|---|---|
| TryHackMe | Lab 01 - Broken Access Control | [Podsumowanie laba](labs/lab-01-tryhackme-broken-access-control.md) |
| TryHackMe | Lab 02 - IDOR | [Podsumowanie laba](labs/lab-02-tryhackme-idor.md) |
| PortSwigger | Lab 03 - User ID controlled by request parameter | [Podsumowanie laba](labs/lab-03-portswigger-user-id-controlled-by-request-parameter.md) |
| PortSwigger | Lab 04 - Insecure direct object references | [Podsumowanie laba](labs/lab-04-portswigger-insecure-direct-object-references.md) |

## Główna zasada

```text
Authenticated nie znaczy authorized.
```

Backend musi sprawdzać, czy obecny użytkownik ma prawo dostać się do konkretnego obiektu albo wykonać konkretną akcję.
