# OS Command Injection Lessons

## The key question is not only whether validation exists

I first suspected the `message` field in a feedback form because free-text fields often have weaker validation. This was not a strong enough reason.

The more important question was:

```text
Which value is likely to reach a process execution API or shell command?
```

The `message` value definitely reached the server because it was present in the HTTP request. What was unknown was whether it reached a command-execution sink.

The `email` field was a better candidate because the feedback feature could plausibly pass it to a mail-related process. That was still a hypothesis until execution evidence confirmed it.

## Facts and assumptions must remain separate

Facts from the blind lab:

- the `email` parameter was user-controlled,
- a shell separator changed execution behaviour,
- a controlled `ping` caused a repeatable delay,
- the delay increased when the packet count increased,
- `whoami` executed and its output was written to a file,
- the file was retrieved through the `/image` endpoint,
- the command ran as a low-privilege lab user named `peter-<random-suffix>`.

Assumptions:

- the original backend command probably related to processing or sending feedback,
- the exact command, executable, and shell were not visible,
- the marketing purpose I initially suggested was not supported by evidence.

## A `500` response does not automatically mean failure

My first timing test returned quickly with:

```text
500 Internal Server Error
"Could not save"
```

That response did not prove command execution. It also did not prove that the input was safe.

After placing a separator on both sides of the injected command, the application returned the same error but waited approximately four seconds for `ping -c 5` and approximately nine seconds for `ping -c 10`.

The important evidence was not the status code. It was the repeatable and proportional timing change.

## The closing separator changed the command structure

I initially described the second separator only as making the request correct. The more accurate explanation is:

```text
original command ; injected command ; remaining original command
```

The closing separator isolated the injected command from fixed text that the backend probably appended after the controlled value.

## Output redirection is an operator, not the output itself

I first described `>` as "the output". The correct explanation is:

```text
whoami > /var/www/images/output.txt
```

`>` redirects standard output from `whoami` into the specified file.

The `/image` endpoint did not execute the command. It only provided a second channel for retrieving the file that the injected command had created.

## Blind does not mean invisible forever

The feedback submission did not return command output directly, so the vulnerability was blind.

Evidence was obtained through:

1. a timing side effect,
2. a filesystem side effect,
3. a separate HTTP request that retrieved the created file.

## Shell execution and process execution are not identical

A process API can start a known executable without passing a command string to a shell. This greatly reduces shell-injection risk, but it does not automatically remove argument injection.

Important questions are:

- Is a shell involved?
- Is the executable fixed?
- Are arguments passed separately?
- Can the user introduce extra flags?
- Does the command support `--` to end option parsing?
- Are values strictly allowlisted?

## Sanitisation is not the primary fix

My first remediation answer focused on validation and sanitisation. This was incomplete.

The preferred order is:

1. do not invoke an OS command when a library or service API can perform the task,
2. if process execution is necessary, use a fixed executable and pass arguments separately without a shell,
3. enforce strict allowlists and types for limited values,
4. consider argument injection and option parsing,
5. apply least privilege, restricted filesystem access, timeouts, output limits, logging, and monitoring,
6. add regression tests that prove shell syntax and option-like input cannot change execution.

## Learning struggle and corrections

- I initially chose `message` based on an assumption about validation rather than tracing likely backend data flow.
- I initially treated a `500` response as the main result instead of separating application errors from execution evidence.
- I learned to use a baseline and then compare controlled timing changes.
- I learned that a separator after the injected command may be needed to isolate it from suffix text.
- I initially described `>` incorrectly and corrected it to stdout redirection.
- My first remediation explanation relied too heavily on sanitisation. I corrected it to prioritise avoiding the shell, safe process APIs, separate arguments, allowlists, least privilege, and regression tests.
