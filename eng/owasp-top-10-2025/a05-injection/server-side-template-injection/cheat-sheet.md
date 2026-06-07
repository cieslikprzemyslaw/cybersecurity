# SSTI Cheat Sheet

> For TryHackMe, PortSwigger, local labs, and explicitly authorised testing only.

## Core model

```text
user input -> template source -> template engine -> evaluated behaviour
```

## First questions

1. What feature generates the output?
2. What exact input do I control?
3. Where does it appear?
4. Is it reflected literally?
5. Is it HTML-encoded?
6. What engine or runtime is suspected?
7. What evidence supports that hypothesis?
8. What single harmless test would prove evaluation?

## Evidence ladder

```text
marker reflected
  -> input control only

expression returns calculated result
  -> template evaluation confirmed

engine-specific error or behaviour
  -> fingerprinting evidence

context/secret/file access
  -> deeper impact confirmed

process execution or destructive side effect
  -> critical impact confirmed
```

## Do not confuse

- Reflection is not SSTI proof.
- HTML rendering is not automatically SSTI.
- SSTI is server-side; XSS is browser-side.
- SSTI leading to a process does not change the root cause into command injection.
- A mathematical result proves evaluation, not full RCE.
- A scanner result is not a substitute for evidence.

## Engine reminders from completed labs

### Smarty

- PHP ecosystem.
- Modifiers/functions depend on security policy and configuration.

### Pug

- Node.js / JavaScript.
- `#{...}` evaluates and escapes output.
- `!{...}` evaluates and produces unescaped output.
- `spawnSync(command, args, options)`.

### Jinja2

- Python-like template language.
- Object traversal chains are environment-dependent.
- `check_output(['ls', '-lah'])` separates executable and arguments.

### ERB

- Ruby expressions.
- A calculated result can confirm evaluation.
- Ruby APIs such as `File` can demonstrate impact in an authorised lab.

## Process API reminders

### Node.js

```javascript
spawnSync('ls', ['-lah'])
```

- Third parameter is an options object.
- `stdout` is a Buffer.

### Python

```python
subprocess.check_output(['ls', '-lah'])
```

- `shell=False` is the default.
- Output is bytes unless text/encoding is requested.

## Root cause statement

> User-controlled input became part of executable template source instead of remaining an ordinary data value passed to a static template.

## Main remediation statement

> Keep template source static and developer-controlled. Pass user input only as template data or variables, and do not recompile or re-evaluate it.

## Testing discipline

Before each test:

- hypothesis,
- reason,
- expected result,
- meaning of a different result.

After each test:

- what changed,
- what stayed the same,
- literal or evaluated,
- errors,
- evidence,
- next hypothesis.
