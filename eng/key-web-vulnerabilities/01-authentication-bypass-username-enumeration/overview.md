# Authentication Bypass / Username Enumeration

## TL;DR

Username enumeration happens when an application reveals whether a username or email address exists. This can happen through different error messages, response lengths, redirects, timing, cookies, or account lockout behaviour.

It does not always give direct access to an account, but it helps attackers focus further attacks on real users.

## What I Practised

I practised three beginner-to-medium authentication labs focused on username enumeration:

- obvious response differences,
- subtle response differences,
- account lockout behaviour.

The goal was not only to complete the labs, but to understand how to compare login responses, isolate variables, and identify signals that reveal valid usernames.

## Key Concept

Authentication bypass and username enumeration are authentication weaknesses. Username enumeration occurs when the login flow gives different feedback depending on whether the username exists.

The difference may be obvious, such as separate messages for invalid usernames and incorrect passwords. It may also be subtle, such as a missing punctuation mark, slightly different response length, or account lockout behaviour that only happens for existing accounts.

## What the User Controls

In these exercises, the main controlled inputs were:

- `username`,
- `password`,
- request body parameters,
- request order and repetition,
- timing and number of login attempts.

In a real application, other controlled inputs may also matter, such as headers, cookies, JSON fields, URL parameters, and password reset form values.

## Where the Input Goes

The username and password are processed by the backend authentication logic.

The backend usually checks:

- whether the username exists,
- whether the password matches,
- whether the account is locked,
- whether rate limits apply,
- whether a session should be created.

If the application returns different responses for different branches of this logic, those differences can leak information.

## Impact

Username enumeration can help an attacker identify real accounts and focus further attacks on them.

Realistic impact includes:

- targeted password guessing,
- password spraying,
- credential stuffing,
- phishing against known users,
- account lockout abuse,
- reduced effort for account takeover attempts.

The issue does not normally expose passwords directly, but it makes other attacks easier and more focused.

## Developer Remediation

Developers should make authentication failure behaviour consistent.

Recommended controls:

- use one generic error message, such as `Invalid username or password`,
- avoid different messages for invalid username and incorrect password,
- keep status codes consistent for failed logins,
- avoid different redirects for valid and invalid usernames,
- avoid obvious response length differences,
- reduce timing differences where possible,
- implement rate limiting consistently,
- design account lockout so it does not confirm account existence,
- avoid exposing lockout messages only for valid users,
- add logging and alerting for suspicious authentication attempts,
- consider MFA for sensitive accounts,
- test login, password reset, and MFA flows with both valid and invalid users.

A useful implementation habit is to use a shared constant or template for failed authentication messages. This helps avoid accidental wording, spacing, or punctuation differences.

## Review Checklist

- Does the login form reveal whether the username exists?
- Are invalid username and invalid password responses identical?
- Are small text differences present in the HTML?
- Are response lengths different?
- Are redirects different?
- Are cookies set differently?
- Does timing differ between valid and invalid usernames?
- Does account lockout trigger only for existing users?
- Can the lockout mechanism be abused to block real users?
- Is rate limiting applied consistently?
- Are failed login attempts logged and monitored?

## Main Takeaway

A login flow should not reveal which part of the authentication attempt was correct. Even tiny differences in error messages, response length, timing, or lockout behaviour can help an attacker identify valid accounts.
