# SSTI Learning Summary

## What I completed

I completed SSTI theory and practical work with four engine/runtime combinations:

- Smarty / PHP in TryHackMe,
- Pug / Node.js in TryHackMe,
- Jinja2 / Python in TryHackMe,
- ERB / Ruby in PortSwigger Web Security Academy.

I also reviewed SSTImap as an automation tool for authorised testing.

## My initial understanding

I understood the general flow:

```text
user-controlled input -> server-side template engine -> evaluated output
```

However, I initially described SSTI too quickly as arbitrary code execution and relied too heavily on "sanitise the input" as the fix.

The corrected model is:

- simple evaluation proves SSTI,
- deeper access must be demonstrated separately,
- the primary fix is to keep user input out of template source.

## Learning struggle: process APIs

### Pug / Node.js

I initially misunderstood `spawnSync()` and tried to:

- pass command plus arguments as one executable string,
- use the third parameter as another command.

I learned:

```javascript
spawnSync(command, args, options)
```

The third parameter is configuration, not a second process. Arguments belong in an array.

### Jinja2 / Python

I saw the same concept with `subprocess.check_output()`:

```python
check_output('ls')
```

can work for a command with no arguments, while:

```python
check_output('ls -lah')
```

usually fails with `shell=False` because the string is treated as the executable name. The correct structure is:

```python
check_output(['ls', '-lah'])
```

This helped me understand the difference between shell parsing and direct process APIs.

## Learning struggle: environment-dependent payloads

At first, long object-traversal chains looked like reusable SSTI syntax.

I learned that paths using:

- Jinja `__mro__` / `__subclasses__()` and hard-coded indexes,
- Node.js `process.mainModule`,

are dependent on versions, imports, runtime mode, available objects, and configuration. They are lab techniques, not universal payloads.

## PortSwigger ERB lab: my reasoning process

### Initial mistake

I first identified `productId` as the controlled parameter because I was looking at product requests.

After examining the out-of-stock flow, I found the actual parameter:

```text
message
```

### Reflection test

I replaced the message with:

```text
SSTI_TEST_123
```

The marker appeared inside a `<div>` in the response.

This proved:

- I controlled `message`,
- I knew where it was rendered.

It did not yet prove SSTI.

### Evaluation test

I used a harmless ERB arithmetic expression. The response contained:

```text
49
```

This proved the server evaluated the expression through ERB rather than displaying it literally.

### File path confusion

I did not know how the target file path had been discovered. I learned that the path was supplied directly in the lab description. This lab tested ERB evaluation and Ruby API use, not filesystem enumeration.

### Ruby knowledge gap

I did not know Ruby's file API. The lab used:

```ruby
File.delete(path)
```

This was a direct Ruby API call and did not require the shell or `rm`.

### Root cause misunderstanding

I initially asked what "root cause" meant.

The final explanation is:

> The application placed the user-controlled `message` value into ERB template source and rendered it. The input became template code instead of remaining data.

### Remediation correction

My first answer focused on sanitisation and cleaning input.

The corrected approach is:

- do not construct template source from user input,
- keep templates static and developer-controlled,
- pass `message` only as a data variable,
- where possible, accept a status ID and select a fixed server-side message.

### Regression testing gap

I initially could not propose a regression test.

I learned a simple test:

```text
send template expression
  -> it must remain text or be rejected
  -> it must not become the calculated result
```

## What I can now explain

- what a server-side template engine does,
- how SSTI differs from reflection, XSS, and OS Command Injection,
- why a calculation proves evaluation but not automatically RCE,
- how error messages and expression behaviour help fingerprint engines,
- why process API argument structure matters,
- why sandboxing is defence in depth,
- why user-controlled data must not become template source,
- how to write a basic SSTI regression test.

## Current assessment

**PASS**

This is a practical and correct foundation for a Frontend Engineer transitioning into AppSec. I do not need another SSTI lab before continuing the A05 learning plan. Additional labs can be used later for revision, sandbox escape concepts, or engine-specific depth.
