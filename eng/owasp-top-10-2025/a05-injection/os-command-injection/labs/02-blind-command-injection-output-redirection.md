# Lab 2 - Blind OS Command Injection with Output Redirection

## Platform

PortSwigger Web Security Academy

Lab: https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection

## Scope

Authorised training lab only.

## Lab goal

Identify the operating-system user running the application when command output is not returned by the vulnerable endpoint.

## Vulnerable feature

```text
POST /feedback/submit
```

Normal form values:

```text
csrf=<token>
name=name
email=email@gmail.com
subject=com
message=message
```

## Initial request discovery

Two requests were first noticed:

```http
GET /product?productId=2
GET /image?filename=44.jpg
```

The image request was interesting because the lab involved output redirection, but it was not the feedback submission sink.

The relevant request was found by submitting the feedback form and inspecting Burp HTTP history:

```http
POST /feedback/submit
```

## Controlled parameters

The form allowed control over:

- `name`,
- `email`,
- `subject`,
- `message`.

The CSRF token was required for the form but was not the suspected command input.

## First assumption

I initially chose `message` because I expected a free-text field to have weak validation.

This reasoning was incomplete. Weak validation does not create Command Injection unless the value reaches a command or process sink.

The `message` value clearly reached the HTTP server because it was present in the request. What was unknown was whether it reached OS command construction.

## Improved hypothesis

The `email` field was a more plausible candidate because a feedback feature might pass an email address to a mail-related command.

This was only a hypothesis. The exact backend command was not visible.

## Normal baseline

Normal feedback requests completed in milliseconds.

## First timing attempt

A benign loopback ping was appended to the email value with one leading separator.

Result:

```text
84 ms
500 Internal Server Error
"Could not save"
```

This did not prove execution.

Possible explanations included:

- command syntax error,
- fixed suffix text becoming arguments to the injected command,
- validation or application failure,
- an incorrect command context.

## Isolating the command

A separator was placed after the injected command as well as before it.

Conceptual structure:

```text
original command ; ping ... ; original suffix
```

The closing separator prevented likely suffix text from becoming part of the injected command.

## Timing evidence

Observed results:

```text
ping -c 5  -> 4,128 ms
ping -c 10 -> 9,256 ms
```

Why this was strong evidence:

- normal requests completed in milliseconds,
- the delay repeated,
- the delay increased with the controlled packet count,
- the response body and status could remain the same while execution time changed.

The `500` response was a clue about application failure, not the proof. The proportional delay was the command-execution evidence.

## Why this was blind

The vulnerable feedback response did not contain stdout from the injected command.

The application returned only:

```text
"Could not save"
```

Execution therefore had to be demonstrated through side effects.

## Output redirection plan

The lab exposed an image endpoint:

```http
GET /image?filename=44.jpg
```

The documented writable directory was:

```text
/var/www/images/
```

A benign command result was redirected to:

```text
/var/www/images/output.txt
```

Conceptual shell fragment:

```text
whoami > /var/www/images/output.txt
```

`>` redirects standard output. It is not the output itself.

## Retrieving the result

The created file was requested through:

```http
GET /image?filename=output.txt
```

Response:

```text
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8

peter-VqYsRp
```

## Evidence chain

```text
controlled email
-> shell separators changed command structure
-> repeatable timing confirmed hidden execution
-> whoami executed
-> stdout redirected to /var/www/images/output.txt
-> /image endpoint retrieved the created file
-> application process user identified
```

## Important distinctions

- `POST /feedback/submit` was the command-injection sink.
- `email` was the confirmed controlled parameter.
- `/image` did not execute the command.
- `/image` provided a separate retrieval channel.
- The exact original backend command was not known.
- The successful Unix-style separator and command suggested a Unix-like shell context, but did not prove the exact shell.

## Root cause

Untrusted email input was allowed to influence a shell-interpreted command string. The application failed to keep user data separate from executable command syntax.

## Real impact

The proof showed arbitrary command execution as the application account. Depending on privileges, an attacker could potentially read files, modify writable content, access secrets, invoke installed tools, disrupt the application, or use allowed network access.

## Remediation

- Replace the shell-based feedback processing with an email library, SDK, service API, or queue.
- If an external process is unavoidable, use a fixed executable and separate argument values with the shell disabled.
- Hardcode command flags and reject option-like input.
- Apply strict email validation as a secondary control, not as the primary fix.
- Run as a dedicated low-privilege account.
- Remove write access to web-readable directories unless explicitly required.
- Apply timeouts, output limits, and monitoring.
- Add behaviour, code-level, filesystem, and egress regression tests.

## Learning struggle and corrections

- I first selected `message` based on validation assumptions rather than likely data flow.
- I initially described the email use as a marketing fact, but the purpose was unsupported and had to remain an assumption.
- My first test used only one separator and returned quickly with `500`.
- I learned that a closing separator can isolate the injected command from backend suffix text.
- I learned not to treat one `500` as proof of success or failure.
- I confirmed execution using repeatable and proportional timing.
- I initially called `>` the output and corrected it to the stdout-redirection operator.
- My first remediation answer relied too heavily on validation and sanitisation.
- I corrected the remediation model to prioritise avoiding the shell and using safe process APIs with separate arguments.

## Knowledge status after debrief

The practical lab was completed successfully. The debrief initially produced a **PARTIAL PASS** because root cause and remediation were not yet explained strongly enough. These notes include the corrected model.

## Regression tests

- `email` values containing separators cannot start additional commands.
- A delay-shaped marker does not increase response time.
- Increasing the marker value does not create proportional delay.
- Input cannot create a file in `/var/www/images/`.
- The application account cannot write to a web-readable directory.
- The feature does not build a shell command string.
- A fixed executable and separate argument list are used if process execution remains necessary.
- Argument values beginning with `-` cannot become options.
- No outbound interaction can be triggered through the form.
