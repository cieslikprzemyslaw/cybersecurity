# Direct and Blind OS Command Injection

## Direct or verbose command injection

Direct command injection returns the output of the injected command in the same HTTP response.

Conceptual flow:

```text
injected command executes
-> stdout is included in application response
-> recognisable output proves execution
```

In the simple PortSwigger lab, the stock-checking response contained both:

- the expected stock value,
- the username returned by `whoami`.

This was direct evidence because the injected command's output appeared in the vulnerable endpoint's response.

## Blind command injection

Blind command injection executes on the server without returning the command output in the vulnerable endpoint's response.

The page may look unchanged, return a generic error, or complete asynchronously.

Blind evidence relies on a side effect or separate observation channel.

## Timing evidence

A timing test should begin with a baseline.

Recommended reasoning:

1. Send the normal request several times.
2. Record typical response time and variation.
3. Change one suspected parameter.
4. Request a controlled delay.
5. Repeat the same test.
6. Change the delay value and check whether the observed time changes proportionally.

Observed lab evidence:

```text
normal request -> milliseconds
ping -c 5     -> approximately 4.1 seconds
ping -c 10    -> approximately 9.3 seconds
```

In this lab environment, the packet count resulted in roughly one second between packets, so the measured delay was close to count minus one. The important point was not an exact formula. It was repeatability and proportional control.

One slow request is not proof because network delay, server load, retries, or unrelated failures can create false positives.

## File-based evidence

If stdout is hidden but a writable and retrievable location exists, output can be redirected in an authorised lab.

Conceptual example:

```text
whoami > /var/www/images/output.txt
```

`>` redirects standard output to a file.

A separate request can then retrieve the file:

```http
GET /image?filename=output.txt
```

Required conditions:

- the application process can write to the chosen directory,
- the retrieval endpoint maps to that directory,
- the command completes,
- the output file is accessible,
- no cleanup removes the file first.

The file endpoint is an observation channel. It is not automatically the command-execution sink.

## Out-of-band evidence

When the command executes asynchronously or neither output nor files are accessible, an authorised OAST service can observe an outbound interaction.

Conceptual channels include:

- DNS lookup,
- HTTP request,
- another controlled network interaction.

OAST evidence should be correlated to a unique marker so the interaction can be connected to one test request.

## Response and status differences

A status-code or body change may be useful, but it is not automatically execution proof.

Example from the blind lab:

```text
500 Internal Server Error
"Could not save"
```

The same error occurred in both fast and delayed requests. The `500` showed that input changed application behaviour, but timing and file retrieval provided the actual command-execution evidence.

## Command separators and isolation

Separators can change a command string into multiple commands.

Common examples:

| Operator | General behaviour |
|---|---|
| `;` | execute the next command regardless of previous success in Unix-like shells |
| `&&` | execute the next command only if the previous command succeeds |
| `||` | execute the next command only if the previous command fails |
| `&` | separator or background operator depending on shell and context |
| `|` | pipe stdout from one command into another |
| newline | command separator in Unix-like shells |

A separator after the injected command may be important:

```text
original command ; injected command ; original suffix
```

Without it, fixed suffix text may become arguments to the injected command or create a syntax error.

PortSwigger's general matrix treats `;` and newline as Unix-only separators, while `&`, `&&`, `|`, and `||` are common across Windows and Unix command contexts. PowerShell has its own syntax and should be assessed separately.

## Quote context

Controlled input may appear inside quotes:

```text
command "USER_INPUT" fixed-suffix
```

In that case, command syntax may remain inside the quoted argument unless the context is first terminated. This is why payload guessing is inefficient. The tester should infer or test the command context one hypothesis at a time.

## Encoding and transport

In `application/x-www-form-urlencoded` data:

- `%3B` decodes to `;`,
- `+` decodes to a space,
- unencoded `&` starts a new form parameter,
- `%26` represents an ampersand inside the value.

Correct transport encoding only ensures that the backend receives the intended characters. It does not make the value safe.

## Evidence hierarchy

From weaker to stronger:

1. generic error or status change,
2. repeatable response difference connected to one hypothesis,
3. repeatable timing change,
4. proportional timing control,
5. unique file or application side effect,
6. direct command output,
7. unique OAST interaction tied to the request.

The strongest evidence is specific, repeatable, controlled, and difficult to explain through normal application behaviour.

## Interview-style takeaway

Direct command injection returns command output in the same response. Blind command injection requires a reliable side channel such as timing, a file, or an outbound interaction. A generic error may be a clue, but evidence should demonstrate changed OS execution rather than only failed validation.
