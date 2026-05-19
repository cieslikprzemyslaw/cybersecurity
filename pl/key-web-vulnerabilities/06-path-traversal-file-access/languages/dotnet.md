# .NET / C# - Path Traversal

## Ryzykowne API

| API | Ryzyko |
|---|---|
| `File.ReadAllText()` | Arbitrary file read |
| `FileStream` | Arbitrary file access |
| `PhysicalFile()` | Może ujawnić pliki, jeśli path jest user-controlled |
| `Path.Combine()` | Samo nie wystarcza |
| download endpoints | Częste cele Path Traversal |

## Niebezpieczny wzorzec

```csharp
var file = Request.Query["file"];
var path = "/app/files/" + file;
return PhysicalFile(path, "application/octet-stream");
```

## Bezpieczniejsze podejście

Gdzie możliwe, używaj IDs albo allowlisted filenames.

Jeśli ścieżki są konieczne, resolve full path i sprawdź, czy zostaje w dozwolonym katalogu.

```csharp
var basePath = Path.GetFullPath("/app/files");
var requested = Path.GetFullPath(Path.Combine(basePath, file));

if (!requested.StartsWith(basePath + Path.DirectorySeparatorChar))
{
    return BadRequest();
}
```

## Notatka dla mnie

`Path.Combine()` tylko składa stringi w ścieżkę. Nie sprawia, że input użytkownika jest bezpieczny.
