# A05: Injection Checklist

Use this checklist during learning, code review, or authorised testing.

## General injection questions

- Does user-controlled input reach an interpreter or processing engine?
- Can the input change the structure of a query, command, template, expression, or instruction?
- Is the input treated as data, or can it become syntax?
- What exact data structure reaches the backend?
- Is it a string, array, object, nested object, JSON value, form value, query parameter, cookie, or header?
- Is validation being used as the only defence?
- Is the secure implementation based on safe APIs, parameterisation, strict schemas, type enforcement, or allowlisted operations?

## SQL Injection checks

- Are SQL queries built by concatenating strings with user input?
- Are URL parameters, request body values, cookies, or headers used in SQL queries?
- Does adding a single quote change the response or cause an error?
- Does adding a SQL comment repair a broken query?
- Can true and false conditions change the response behaviour?
- Can `UNION SELECT` return additional data in the response?
- Can blind SQLi be detected through response differences or timing?
- Are database errors exposed to the user?

## NoSQL Injection checks

### Input and data structure

- Does the endpoint expect a primitive string but accept an object or array?
- Can JSON input contain nested objects where a string is expected?
- Can URL-encoded form notation such as `field[operator]=value` become a nested server-side object?
- Does a body parser preserve attacker-controlled keys beginning with `$`?
- Can a user control query field names, operators, sorting, projection, or filters?

### Operator Injection

- Can input become an operator object such as `$ne`, `$nin`, `$gt`, or `$regex`?
- Does changing a value into a nested object alter authentication or lookup behaviour?
- Can the query return a user without validating the supplied credential?
- Can regex conditions create a true/false oracle for secret extraction?

### Syntax Injection

- Is user input concatenated into `$where` or another custom JavaScript expression?
- Does a single quote cause an error or change behaviour?
- Do controlled true and false conditions produce different responses?
- Can an always-true condition return records outside the intended filter?
- Are verbose MongoDB or JavaScript interpreter errors exposed?

### Attack-surface discovery

- Is the visible browser URL the real data endpoint?
- Does the page make background fetch/XHR requests?
- Is the vulnerable request visible only in Burp HTTP history or browser DevTools?
- Are redirects hiding the endpoint that actually processes the data?

## OS Command Injection checks

### Data flow and execution context

- Which request parameter, body field, header, cookie, filename, or route value is user-controlled?
- Where does that value go after the HTTP request is parsed?
- Does it reach a process execution API?
- Is the executable fixed by the developer, or can the client influence it?
- Is a complete command built as one concatenated string?
- Is a shell or command interpreter explicitly invoked?
- Does the code use `shell: true`, `sh -c`, `bash -c`, `cmd /c`, or PowerShell to interpret a string?
- Are command and arguments passed separately?
- Can the user introduce extra flags even when a shell is not involved?
- Can an end-of-options delimiter such as `--` be used where supported?

### Evidence and controlled testing

- What is the normal request and normal response time?
- Does a benign marker appear directly in the response?
- Does a controlled time delay repeat and scale with the chosen value?
- Does the test create a predictable file or another non-destructive side effect?
- Can the application make a controlled outbound request in an authorised OAST lab?
- Did the response status change without proving execution?
- Is a `500` response only an application error, or is there independent evidence of execution?
- Are operating-system assumptions based on evidence or only on payload behaviour?
- Does a successful separator show that a shell-like interpreter is involved, without proving the exact shell?

### Command context

- Is the controlled value inside single or double quotes?
- Is input placed before or after fixed command arguments?
- Does a closing separator isolate the injected command from suffix text?
- Could unencoded `&` split form or query parameters before the backend receives the intended value?
- Is the input decoded once or more than once before execution?
- Are line breaks, pipes, conditional operators, substitutions, or redirection interpreted by the shell?

### Direct and blind behaviour

- Does command output return in the same HTTP response?
- If output is hidden, can timing provide repeatable evidence?
- Can stdout be redirected to a writable and web-readable location in a lab?
- Is an external interaction available for asynchronous or non-readable output?
- Is the observed side effect caused by the injected command rather than normal application behaviour?

## Server-Side Template Injection checks

### Data flow and rendering context

- Which feature compiles or renders server-side templates?
- Can request, stored, CMS, profile, document, email, or status-message data reach template source?
- Is user input passed as a variable to a static template, or concatenated into a template string?
- Can a user control template delimiters, expressions, helper names, filters, or partial names?
- Is any value rendered once and then evaluated again by a second template pass?

### Evidence and fingerprinting

- Does a unique marker appear in the response, and where?
- Does reflection prove only input control rather than template evaluation?
- Does a harmless engine-specific expression return a calculated result?
- Do verbose errors reveal the engine, runtime, template path, parser token, or stack trace?
- Are engine assumptions supported by runtime evidence, source code, package files, or documented architecture?
- Is a generic `500` response treated as a clue rather than proof?

### Impact and containment

- What globals, locals, helpers, filters, plugins, objects, and functions are exposed to the template?
- Can the template reach configuration, secrets, filesystem APIs, database clients, network clients, or process APIs?
- Are file, network, or process side effects tested only in authorised labs or instrumented test environments?
- Are application process permissions limited enough to reduce impact?
- Is a template sandbox present only as defence in depth, and is its policy reviewed?

## Cross-Site Scripting checks

- Are all attacker-controlled values traced from source to browser sink?
- Is output encoding appropriate for the final HTML, attribute, JavaScript, CSS, or URL context?
- Is raw HTML sanitised with an allowlist-based maintained library?
- Are React escape hatches such as `dangerouslySetInnerHTML` reviewed?
- Are CMS rich-text fields treated according to their real trust level?
- Are dangerous URL schemes rejected?
- Are reflected, stored, and DOM-based paths tested?
- Is CSP used only as defence in depth rather than the primary fix?

## AI Prompt Injection awareness checks

- Does the feature use an LLM, agent, RAG, or external-content ingestion?
- Are system/developer instructions separate from user-controlled input?
- Is retrieved content clearly treated as untrusted data?
- Is access control enforced before retrieval?
- Are tools limited by least privilege?
- Are tool arguments validated outside the model?
- Are high-risk actions approved by a human?
- Is model output treated as untrusted before downstream use?
- Are direct and indirect Prompt Injection tests included?
- Does reporting distinguish model output from verified execution and impact?

## Evidence to look for

- database or interpreter error messages,
- HTTP `500` or another controlled error after syntax-sensitive input,
- different response body for true versus false conditions,
- a lookup returning a different user,
- additional products or records appearing,
- a response-length or marker difference,
- a secret length confirmed through repeated conditions,
- one successful character result per secret position,
- direct command output in the HTTP response,
- repeatable and proportional response delay,
- a predictable file created by command output redirection,
- an authorised outbound DNS or HTTP interaction,
- a template expression replaced with a calculated result,
- a template-engine error or stack trace tied to controlled input,
- a stored value evaluated later by a server-side template,
- browser-side script execution or a meaningful DOM/browser side effect,
- model behaviour changed by user or retrieved content,
- a verified LLM tool call, data access, or state change,
- process, filesystem, or application-state changes that were not part of the intended feature.

## Secure implementation checks

- Prepared statements or parameterised APIs are used for SQL.
- NoSQL inputs are validated against strict schemas.
- Authentication fields are enforced as primitive strings.
- Nested objects and operator-shaped input are rejected where not required.
- User-controlled values are not inserted into `$where` or custom JavaScript.
- Built-in structured query filters are preferred over custom query expressions.
- Passwords are verified using password-hash verification, not queried as plaintext values.
- OS commands are avoided when a language library or service API can perform the task.
- User input is never concatenated into a shell command string.
- A fixed executable and separate argument list are used when process execution is unavoidable.
- Shell execution is disabled unless there is a reviewed and documented reason for it.
- Commands, flags, and limited-value arguments use strict allowlists.
- Argument injection is considered even when shell metacharacters are not interpreted.
- The application process follows least privilege.
- Writable directories are restricted and web roots are not unnecessarily writable.
- Process execution has timeouts, output limits, and resource controls.
- Suspicious execution failures, unusual delays, and unexpected child processes are logged and monitored.
- Server-side templates are static and developer-controlled.
- User-controlled values are passed only as template data, variables, or locals.
- Dynamic template compilation from request or stored user data is avoided.
- Template contexts expose only the minimum required objects and helpers.
- User-authored templates, where intentionally supported, use sandboxing, allowlists, quotas, and isolation.
- Browser output uses context-aware encoding and safe DOM APIs.
- Raw HTML is sanitised with a maintained allowlist-based HTML sanitiser when it is genuinely required.
- LLM and agent features enforce authorization, retrieval scope, tool permissions, and high-risk approvals outside the model.
- Model-generated HTML, SQL, shell content, or tool arguments are treated as untrusted before downstream use.
- Production does not expose verbose database, interpreter, process, or stack-trace errors.
- Regression tests cover strings, objects, template syntax, shell syntax, command flags, timing, and side effects.

## Frontend/AppSec angle

- Do not assume values are safe because the frontend generated the link or form.
- Do not assume hidden fields, cookies, predefined categories, or client validation are trusted.
- Do not rely on GET versus POST for security.
- Inspect background API requests, not only the address bar.
- A frontend email validator does not protect a backend process-execution sink.
- URL encoding changes transport representation, not trust.
- The security boundary is how the backend parses input and constructs the query, command, or argument list.

For longer topic-specific checklists, use:

- [SQL Injection cheat sheet](sql-injection/cheat-sheet.md)
- [NoSQL Injection cheat sheet](nosql-injection/cheat-sheet.md)
- [OS Command Injection cheat sheet](os-command-injection/cheatsheet.md)
- [Server-Side Template Injection cheat sheet](server-side-template-injection/cheat-sheet.md)
- [XSS testing cheatsheet](xss-injection-mapping/testing-cheatsheet.md)
- [LLM01 Prompt Injection checklist](../../owasp-top-10-for-llm-applications-2025/llm01-prompt-injection/awareness-checklist.md)
