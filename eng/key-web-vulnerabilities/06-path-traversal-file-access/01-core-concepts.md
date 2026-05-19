# Core Concepts - Path Traversal / File Access Bugs

## What is Path Traversal?

Path Traversal is a vulnerability where user-controlled input influences a server-side file path and the backend reads a file outside the intended directory.

The important point is that the attacker is not browsing the server like using `cd` in a terminal. Instead, the attacker manipulates a path in the request, and the application reads the file on their behalf.

## File name vs file path

A file name is just the name of a file:

```txt
avatar.jpg
invoice.pdf
config.php
```

A file path describes where the file is located:

```txt
/var/www/images/avatar.jpg
../../config.php
/etc/passwd
```

A safer design should avoid letting users control raw filesystem paths.

## Base directory

A base directory is the directory where the application expects files to come from.

Example:

```txt
/var/www/images/
```

The application may intend to read only images from that directory.

Vulnerable pattern:

```txt
/var/www/images/ + userInput
```

If `userInput` is `../../../etc/passwd`, the final resolved path may escape the images directory.

## Relative path

A relative path depends on the current directory or base directory.

Examples:

```txt
avatar.jpg
../config.php
../../etc/passwd
```

## Absolute path

An absolute path starts from the root of the filesystem.

Linux examples:

```txt
/etc/passwd
/var/www/images/avatar.jpg
```

Windows examples:

```txt
C:\Windows\win.ini
C:\Users\Public\file.txt
```

In some cases, an application may block `../` but still accept an absolute path such as `/etc/passwd`.

## What does `../` mean?

`../` means "go one directory up".

Example:

```txt
/var/www/images/../
```

resolves to:

```txt
/var/www/
```

Multiple `../` sequences can move up multiple levels.

## Canonical / resolved path

A canonical or resolved path is the final real path after the filesystem normalizes traversal sequences such as `../`, `./` and duplicated slashes.

Example:

```txt
/var/www/images/../../../etc/passwd
```

can resolve to:

```txt
/etc/passwd
```

This matters because security checks should validate the final resolved path, not just the raw input string.

## Why weak string checks fail

A check such as:

```txt
Does the input start with /var/www/images/?
```

is not enough if the application does not check the final resolved path.

Example:

```txt
/var/www/images/../../../etc/passwd
```

This starts with `/var/www/images/`, but the final path may point to `/etc/passwd`.

## Mental model

```txt
request -> parameter -> backend file logic -> filesystem -> response
```

Path Traversal happens when user-controlled input reaches file access logic without proper validation of the final resolved path.
