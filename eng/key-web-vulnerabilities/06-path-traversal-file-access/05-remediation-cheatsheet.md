# Fixes / Review Cheatsheet

## Core rule

Never let user input directly decide which server-side file is read or included.

## Things to check

| Measure | What it prevents |
|---|---|
| Use file IDs instead of filenames | Prevents direct path manipulation |
| Use allowlist mappings | Prevents arbitrary file access |
| Resolve canonical path | Detects final path after traversal normalization |
| Verify path stays inside base directory | Prevents directory breakout |
| Disable remote includes | Prevents RFI |
| Disable unnecessary wrappers | Reduces wrapper-based abuse |
| Disable verbose errors in production | Prevents path disclosure |
| Use least privilege | Limits impact of file read bugs |
| Keep runtime/framework updated | Removes known historical bypasses |
| Add regression tests | Prevents the bug from returning |

## Bad fix

```js
filename.replace("../", "")
```

This is weak because traversal can be nested, encoded, double encoded or reconstructed after filtering.

## Safer pattern

1. Do not accept raw paths from users.
2. Use IDs or allowlisted keys.
3. Map user choices to known server-side files.
4. Resolve the final path.
5. Check that the final path stays inside the allowed base directory.
6. Enforce authorization.
7. Return generic errors.
8. Add regression tests for known bypasses.

## Allowlist / mapping example

Instead of:

```php
include($_GET["page"]);
```

Use:

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

The user controls a safe key, not the actual filesystem path.

## Canonical path validation

The application should verify the final resolved path.

Conceptual example:

```txt
base directory: /var/www/images/
requested path: /var/www/images/../../../etc/passwd
resolved path: /etc/passwd
```

The request should be blocked because the resolved path no longer stays inside `/var/www/images/`.

## Regression test ideas

- `../../../etc/passwd`
- `/etc/passwd`
- `....//....//etc/passwd`
- `%2e%2e%2f`
- `..%252f..%252fetc%252fpasswd`
- `/var/www/images/../../../etc/passwd`
