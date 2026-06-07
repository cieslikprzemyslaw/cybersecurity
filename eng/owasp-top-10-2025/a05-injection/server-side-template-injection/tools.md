# Tools

## SSTImap

Official repository:

- https://github.com/vladko312/SSTImap

SSTImap is an automated tool for detecting and assessing Server-Side Template Injection and related code-injection behaviour across multiple template engines.

Potential uses in authorised environments include:

- finding injectable parameters,
- identifying likely template engines,
- testing render, error-based, and blind behaviours,
- assessing possible capabilities such as code evaluation or file access.

## Manual-first rule

Use SSTImap after forming and testing a manual hypothesis.

```text
identify feature
  -> identify controlled input
  -> prove reflection
  -> manually test evaluation
  -> form engine hypothesis
  -> use SSTImap as supporting verification
  -> manually confirm important findings
```

Do not treat every tool line as an unquestionable fact. Verify:

- injection point,
- engine identification,
- response evidence,
- operating system assumptions,
- reported capabilities,
- whether a side effect actually occurred.

## Version awareness

Command-line options can change between versions. Always check the installed version:

```bash
python3 sstimap.py -h
```

A command copied from a TryHackMe AttackBox may not use the same flags as the newest GitHub version.

## Scope and safety

Use SSTImap only against:

- TryHackMe,
- PortSwigger Web Security Academy,
- local labs,
- explicitly authorised targets.

Do not use automation to test real systems without permission.

## What the tool should not replace

SSTImap does not replace the ability to explain:

- what input is controlled,
- where it is rendered,
- what evidence proves evaluation,
- why a specific engine is suspected,
- what the root cause is,
- how to remediate the issue.
