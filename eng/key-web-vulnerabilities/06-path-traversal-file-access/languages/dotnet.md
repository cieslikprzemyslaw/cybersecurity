# .NET / C# - Path Traversal

## Risky APIs

| API | Risk |
|---|---|
| `File.ReadAllText()` | Arbitrary file read |
| `FileStream` | Arbitrary file access |
| `PhysicalFile()` | Can disclose files if path is user-controlled |
| `Path.Combine()` | Not enough by itself |
| download endpoints | Common target for path traversal |

## Dangerous pattern

```csharp
var file = Request.Query["file"];
var path = "/app/files/" + file;
return PhysicalFile(path, "application/octet-stream");
```

## Safer approach

Use IDs or allowlisted filenames where possible.

If paths are needed, resolve the full path and verify it stays in the intended directory.

```csharp
var basePath = Path.GetFullPath("/app/files");
var requested = Path.GetFullPath(Path.Combine(basePath, file));

if (!requested.StartsWith(basePath + Path.DirectorySeparatorChar))
{
    return BadRequest();
}
```

## Note to self

`Path.Combine()` only combines strings into a path. It does not make user input safe.
