# Detection and Fingerprinting

## Manual-first workflow

### 1. Identify the feature

Look for features that generate server-side output from user-controlled values, for example:

- notification messages,
- email previews,
- CMS templates,
- user-customisable themes,
- invoice or document generation,
- product or profile pages,
- error pages,
- saved templates.

### 2. Identify controlled input

Possible sources include:

- query parameters,
- form fields,
- JSON properties,
- headers,
- cookies,
- stored CMS fields,
- profile values,
- imported template content.

Do not assume the most obvious ID parameter is the injection point.

### 3. Prove reflection first

Use a unique marker:

```text
SSTI_TEST_123
```

Record:

- whether it appears,
- where it appears,
- whether it is encoded,
- whether it is changed,
- whether it appears immediately or later.

Reflection proves input control, not template evaluation.

### 4. Test one engine hypothesis

Use one simple, non-destructive expression appropriate to the suspected engine.

Before testing, define:

- what behaviour is being tested,
- why the engine is suspected,
- expected result if evaluated,
- expected result if displayed literally,
- what an error would suggest.

### 5. Compare the result

Possible outcomes:

| Result | Meaning |
|---|---|
| Literal expression | No evaluation, wrong syntax, wrong context, or escaping before compilation |
| Calculated result | Strong evidence of server-side template evaluation |
| Engine-specific error | Useful fingerprinting clue |
| Generic server error | Indicates processing failed, but does not identify the engine alone |
| Changed timing or side effect | May support blind evaluation, but requires repeatable evidence |

## Fingerprinting sources

### Syntax differences

Engines use different delimiters and expression rules. Similar syntax does not guarantee the same engine.

### Type behaviour

Different engines may treat numbers and strings differently. A deliberately chosen harmless expression may distinguish engines based on output.

### Error messages

Verbose errors may reveal:

- engine names,
- template filenames,
- language/runtime stack traces,
- parser tokens,
- line numbers,
- framework paths.

Errors are clues, not permission to assume every part of the environment.

### Application stack

Headers, framework behaviour, source code, package files, and documented architecture can support an engine hypothesis. They should be combined with runtime evidence.

## Avoid payload guessing

Bad workflow:

```text
try Jinja syntax
try Twig syntax
try Pug syntax
try ERB syntax
try random payload list
```

Better workflow:

```text
feature
  -> controlled input
  -> reflected location
  -> suspected processing path
  -> one engine hypothesis
  -> one controlled test
  -> interpret evidence
```

## Completed lab lesson

In the PortSwigger ERB lab, I initially thought `productId` was the controlled SSTI input because it appeared in product requests.

The actual flow was:

```text
out-of-stock product
  -> redirect / request containing message parameter
  -> message rendered on the home page
```

A marker in `message` proved reflection. Only the ERB arithmetic test returning `49` proved evaluation.

## What does not matter by itself?

- The HTTP method alone.
- A parameter name that sounds important.
- HTML appearing in a response.
- One generic `500` response.
- A copied payload without understanding its engine or context.
- A tool reporting an engine without manual verification.
