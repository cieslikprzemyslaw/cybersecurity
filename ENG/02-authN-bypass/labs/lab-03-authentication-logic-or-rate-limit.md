# Lab 03 - Username Enumeration via Account Lock

Topic: Authentication Bypass / Username Enumeration  
Focus: Account lockout behaviour as an enumeration signal

## Goal

The goal of this lab was to understand how account lockout, which is normally a security control, can accidentally reveal whether a username exists.

## Input I Controlled

I controlled:

```text
username
password
number of repeated login attempts
request order
```

The main testing behaviour was sending several wrong passwords for the same username and comparing how the application responded.

## What Happened

For random or non-existing usernames, repeated failed login attempts continued to return the normal generic error message.

For one candidate username, repeated failed attempts triggered an account lockout message. This showed that the application was tracking failed attempts for that username, which indicated that the account existed.

After identifying the valid username, password testing showed one response that behaved differently from normal failed attempts. After the lockout period expired, the same credentials allowed a successful login.

Important response signals:

- normal failed attempts returned `200 OK`,
- random usernames kept returning a generic error,
- one username triggered a lockout message after repeated attempts,
- one password attempt produced a different response pattern,
- successful login produced a redirect to the account page and a new session cookie.

## What I Learned

I learned that security controls can also leak information if they are implemented inconsistently.

Account lockout is useful, but if lockout appears only for existing accounts, attackers can use it to enumerate usernames.

I also learned that this type of issue may not be visible from a single request. It requires testing behaviour over several attempts.

## Impact

In a real application, attackers could use account lockout behaviour to identify valid users.

The issue may support:

- password spraying,
- credential stuffing,
- targeted brute-force attempts,
- phishing against known users,
- account lockout abuse or denial of service against real users.

## Remediation Summary

See `../authn-bypass-username-enumeration-overview.md` for the general remediation section.

Lab-specific remediation:

- do not expose lockout messages only for existing usernames,
- keep authentication failure responses consistent,
- apply rate limiting consistently across valid and invalid usernames,
- combine account-based, IP-based, device-based, and behavioural controls carefully,
- avoid allowing attackers to easily lock out other users,
- log and monitor repeated failed login attempts,
- alert on password spraying and credential stuffing patterns,
- consider MFA for sensitive accounts.

## Main Takeaway

Account lockout is a useful defence, but if it behaves differently for existing and non-existing users, it becomes a username enumeration signal.
