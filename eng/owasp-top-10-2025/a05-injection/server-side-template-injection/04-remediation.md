# SSTI Remediation

## Primary fix

Never allow user-controlled content to become template source.

```text
secure:
developer-controlled static template
  + user-controlled values passed as data

insecure:
static text + user input
  -> dynamic template compilation
```

## Secure design principles

### 1. Keep templates static and developer-controlled

Store templates in reviewed files or controlled resources. Do not concatenate query parameters, form values, CMS fields, or profile values into template source.

### 2. Pass input only as variables or locals

The engine should receive a trusted template and a separate data object.

Conceptual example:

```text
render(template='out-of-stock', data={ message: safeMessage })
```

The variable value must not be reparsed as a second template.

### 3. Prefer identifiers over arbitrary display text

Instead of accepting a full message from the client:

```text
?message=Unfortunately this product is out of stock
```

accept a limited identifier:

```text
?status=out_of_stock
```

The server selects the final message from a fixed mapping or translation catalogue.

### 4. Avoid dynamic compilation APIs

Review calls that compile or render templates from strings, especially when the string contains request data.

Examples of risky patterns include conceptually:

- `from_string(userControlledValue)`
- `compile(prefix + userControlledValue)`
- `render(userControlledTemplateSource)`

The exact API differs by framework and language.

### 5. Restrict the template context

Expose only the minimum data and helper functions needed by the template.

Do not expose:

- process objects,
- environment objects,
- filesystem helpers,
- database clients,
- powerful service containers,
- secrets or configuration objects.

### 6. Sandbox only as defence in depth

A sandbox can restrict attributes, methods, functions, file access, or language features when untrusted users must be allowed to author templates.

It is not the primary fix for an application that never needed user-authored templates.

Sandbox risks include:

- incomplete policies,
- dangerous allowed helpers,
- version-specific bypasses,
- excessive objects in the template context,
- denial-of-service through expensive expressions.

### 7. Use output escaping for XSS, not as the SSTI fix

HTML escaping protects the generated output context. It does not prevent a server-side expression from being evaluated before output is escaped.

### 8. Apply least privilege

Run the application with only the filesystem, network, and operating-system permissions it needs.

This reduces impact if a template bug reaches file or process APIs.

### 9. Avoid verbose production errors

Return generic errors to users and keep detailed template exceptions in protected server-side logs.

### 10. Maintain and review template engines

Keep the engine and framework patched. Review security configuration, allowed plugins, functions, modifiers, globals, and custom extensions.

## Engine-specific defence-in-depth notes

### Jinja2

- Use static templates and data variables first.
- When user-authored templates are an intentional feature, consider `SandboxedEnvironment`.
- Restrict globals, filters, tests, and callable objects.
- Remember that autoescape primarily helps with XSS output handling.

### Pug

- Do not compile user-controlled strings as Pug source.
- Pass values as locals to a static template.
- Prefer escaped output for untrusted values.
- Avoid exposing powerful Node.js runtime objects or process helpers.

### Smarty

- Enable and configure an appropriate security policy when untrusted templates are an intentional feature.
- Allowlist safe modifiers, functions, plugins, and resources.
- Do not treat disabling one tag as a complete security boundary.

### ERB

- Do not build an ERB template by concatenating request values.
- Render static `.erb` templates with variables supplied separately.
- Do not expose arbitrary Ruby objects or helper methods to untrusted template authors.

## Why "sanitize the input" is not enough

A blacklist of delimiters or keywords is fragile because:

- engines support multiple syntax forms,
- encodings and transformations may change input,
- context affects parsing,
- valid text may be broken,
- future engine features may create new bypasses.

Validation is useful for business rules. The security boundary must be the separation of template source and data.

## Code review questions

- Is any request value used to create template source?
- Does the application compile templates from strings?
- Can stored CMS or profile content become template syntax?
- Is any rendered value evaluated more than once?
- Which objects and helpers are available inside the template?
- Are users intentionally allowed to author templates?
- If yes, what sandbox, allowlist, quotas, and isolation controls exist?
- What filesystem and network permissions does the process have?
