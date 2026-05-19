# PHP - Path Traversal / File Inclusion

## Risky APIs and features

| Function / feature | Risk |
|---|---|
| `include()` | Can include user-controlled local or remote files |
| `require()` | Can include user-controlled local or remote files |
| `include_once()` | Same as include, but only once |
| `require_once()` | Same as require, but only once |
| `file_get_contents()` | Can read arbitrary files if path is user-controlled |
| `readfile()` | Can disclose file contents |
| `fopen()` | Can open user-controlled paths |
| `php://filter` | Can expose source code or transform streams |
| `data://` | Can provide inline data as a stream |
| `allow_url_include` | Can enable RFI |

## Dangerous pattern

```php
include($_GET["page"]);
```

The user controls which file PHP includes.

## Safer pattern

```php
$allowed = [
    "en" => "languages/EN.php",
    "ar" => "languages/AR.php"
];

$lang = $_GET["lang"] ?? "en";

if (array_key_exists($lang, $allowed)) {
    include($allowed[$lang]);
}
```

## Note to self

PHP include patterns are especially sensitive because file access can become file execution.
