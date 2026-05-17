# Authentication Bypass / Username Enumeration - Practical Work Cheat Sheet

## Purpose

Use this checklist when testing authentication flows in legal labs, training environments, or systems where you have explicit permission.

The goal is to identify whether the application reveals useful information during login, password reset, account lockout, or other authentication-related flows.

## What to Check During Login Testing

Start with a normal login attempt and capture the request in Burp Suite.

Check:

- endpoint, for example `/login`, `/signin`, or `/auth`,
- HTTP method,
- request body format,
- parameters you control,
- cookies and session behaviour,
- response status code,
- response body,
- response length,
- redirects,
- timing,
- rate limiting,
- account lockout behaviour.

Common controlled inputs:

- username,
- email address,
- password,
- remember-me value,
- CSRF token,
- headers,
- cookies,
- JSON body fields,
- URL parameters.

## Username Enumeration Signals

A username enumeration issue may exist if the application behaves differently depending on whether the username exists.

Look for differences in:

- visible error messages,
- punctuation, spaces, or small text changes,
- HTTP status code,
- `Location` header,
- `Set-Cookie` header,
- response length,
- response timing,
- account lockout messages,
- rate-limit behaviour,
- password reset messages,
- MFA prompts,
- email verification flows.

Examples of risky behaviour:

- `Invalid username` for unknown users,
- `Incorrect password` for valid users,
- lockout message only for real accounts,
- slower response only for real accounts,
- MFA step only after entering a valid username,
- password reset message that confirms whether an email exists.

## Response Comparison Checklist

When comparing valid and invalid attempts, check:

- Is the response text exactly the same?
- Is the punctuation identical?
- Is the HTML structure identical?
- Is the response length the same or very close?
- Is the status code the same?
- Are redirects the same?
- Are cookies set in the same way?
- Does one request take longer than the other?
- Does the account lockout trigger only for some usernames?
- Does rate limiting apply consistently?

Do not rely only on what you can see in the browser. Small differences in raw HTML can reveal useful information.

## Burp Repeater Practical Flow

1. Log in with a random username and random password.
2. Send the login request to Repeater.
3. Change only one value at a time.
4. Compare responses for:
   - unknown username + wrong password,
   - possible valid username + wrong password,
   - valid username + candidate password.
5. Record any difference in status, length, body, redirect, cookie, or timing.
6. If a response looks different, confirm it with another request before treating it as a real signal.

Good testing habit:

```text
Change one variable at a time.
```

If both username and password change at the same time, it becomes harder to understand what caused the response difference.

## Burp Intruder Practical Flow

Use Intruder when you need to test many usernames or passwords.

Recommended flow:

1. Test usernames first:

```text
username=§candidate_username§&password=FixedWrongPassword123
```

2. Look for differences in:

```text
Status | Length | Grep - Extract | Redirect | Set-Cookie
```

3. After finding a likely valid username, test passwords:

```text
username=KnownCandidate&password=§candidate_password§
```

4. Look for a successful login signal, such as:

```text
302 redirect to /my-account
new session cookie
missing error message
changed response length
```

## Useful Burp Features

### Grep - Extract

Use this to extract the warning message from the response, for example the text inside:

```html
<p class=is-warning>...</p>
```

This makes it easier to compare many responses in Intruder without reading the full HTML manually.

### Grep - Match

Use this to search for specific phrases, such as:

```text
Invalid username or password
Too many incorrect login attempts
```

### Comparer

Use Comparer to compare two suspicious responses. This is helpful when the difference is only a punctuation mark, space, or small HTML change.

### Resource Pool

For account lockout or timing tests, use a single-threaded resource pool when order matters.

```text
1 concurrent request
```

This makes results easier to interpret.

## Account Lockout Testing

Account lockout is a security control, but it can become a username enumeration signal if it behaves differently for valid and invalid usernames.

Check:

- Does lockout happen only for existing usernames?
- Does the message reveal that an account exists?
- Does the lockout apply by username, IP, session, or device?
- Can an attacker lock other users out of their accounts?
- Is the lockout duration shown to the user?
- Is logging and alerting in place?

Testing pattern:

```text
valid-looking username + several wrong passwords
random username + several wrong passwords
```

Compare the behaviour.

## Common Mistakes

Avoid these mistakes:

- changing username and password at the same time,
- relying only on the browser view,
- ignoring response length,
- ignoring small punctuation differences,
- ignoring redirects and cookies,
- assuming generic messages are always safe,
- forgetting that lockout behaviour can leak valid usernames,
- testing too aggressively without understanding rate limits,
- publishing passwords, tokens, cookies, or direct lab answers in GitHub notes.

## Developer Remediation

Developers should:

- use one generic authentication failure message,
- keep the same visible response for valid and invalid usernames,
- avoid different status codes for username-related failures,
- avoid different redirects for valid and invalid usernames,
- avoid response length differences where possible,
- avoid timing differences where possible,
- implement rate limiting consistently,
- design account lockout carefully,
- avoid lockout messages that confirm account existence,
- log suspicious authentication activity,
- alert on brute-force and credential stuffing patterns,
- consider MFA for sensitive accounts,
- test authentication flows with valid and invalid inputs.

A practical developer habit:

```text
Use one shared constant or template for authentication failure messages.
```

This reduces the chance of tiny differences such as missing punctuation or inconsistent wording.

## Safe and Legal Testing Reminder

Only test authentication flows where you have permission. Brute-force testing against real systems can be illegal, disruptive, and harmful.

For real applications, use agreed test accounts, approved scope, rate limits, and documented testing windows.

## Review Questions

- What parameters do I control in the login request?
- Does the application respond differently for valid and invalid usernames?
- Is the error message truly identical?
- Are status codes and redirects consistent?
- Does response length reveal anything?
- Does timing reveal anything?
- Does account lockout reveal whether an account exists?
- Can the lockout be abused to block real users?
- Is rate limiting applied consistently?
- Would a developer understand how to fix this from my report?

## Main Takeaway

Authentication testing is not only about finding the correct password. It is about understanding how the application behaves when authentication fails. Any consistent difference can become a signal for attackers.
