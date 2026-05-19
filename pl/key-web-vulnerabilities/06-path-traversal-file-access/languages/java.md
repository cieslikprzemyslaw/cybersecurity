# Java - Path Traversal

## Ryzykowne API

| API | Ryzyko |
|---|---|
| `File` | Może wskazać dowolną ścieżkę |
| `FileInputStream` | Może czytać dowolne pliki |
| `Paths.get()` | Może budować ścieżki z inputu użytkownika |
| `Files.readAllBytes()` | Może ujawnić pliki |
| download servlets/controllers | Częste cele Path Traversal |

## Niebezpieczny wzorzec

```java
String file = request.getParameter("file");
Path path = Paths.get("/app/files/" + file);
byte[] data = Files.readAllBytes(path);
```

## Bezpieczniejsze podejście

```java
Path baseDir = Paths.get("/app/files").toRealPath();
Path requested = baseDir.resolve(file).normalize().toRealPath();

if (!requested.startsWith(baseDir)) {
    throw new SecurityException("Invalid file path");
}
```

Dodatkowo egzekwuj authorization przed zwróceniem pliku.

## Notatka dla mnie

W Javie ryzykowny moment to zwykle budowanie `Path` albo `File` bezpośrednio z danych requestu.
