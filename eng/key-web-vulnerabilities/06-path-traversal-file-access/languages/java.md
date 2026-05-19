# Java - Path Traversal

## Risky APIs

| API | Risk |
|---|---|
| `File` | Can point to arbitrary paths |
| `FileInputStream` | Can read arbitrary files |
| `Paths.get()` | Can build paths from user input |
| `Files.readAllBytes()` | Can disclose files |
| download servlets/controllers | Common path traversal targets |

## Dangerous pattern

```java
String file = request.getParameter("file");
Path path = Paths.get("/app/files/" + file);
byte[] data = Files.readAllBytes(path);
```

## Safer approach

```java
Path baseDir = Paths.get("/app/files").toRealPath();
Path requested = baseDir.resolve(file).normalize().toRealPath();

if (!requested.startsWith(baseDir)) {
    throw new SecurityException("Invalid file path");
}
```

Also enforce authorization before returning the file.

## Note to self

In Java, the dangerous step is usually building a `Path` or `File` directly from request data.
