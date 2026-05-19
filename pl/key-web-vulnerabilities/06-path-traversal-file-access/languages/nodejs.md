# Node.js - Path Traversal

## Ryzykowne API

| API | Ryzyko |
|---|---|
| `fs.readFile()` | Arbitrary file read |
| `fs.createReadStream()` | Arbitrary file streaming |
| `res.sendFile()` | File disclosure, jeśli path jest user-controlled |
| `path.join()` | Przydatne, ale samo nie wystarcza |
| `path.resolve()` | Przydatne tylko z base directory validation |
| static file middleware | Zła konfiguracja może ujawnić pliki |

## Niebezpieczny wzorzec

```js
app.get("/download", (req, res) => {
  res.sendFile("/app/files/" + req.query.file);
});
```

## Bezpieczniejsze podejście

```js
import path from "path";

const baseDir = path.resolve("/app/files");

app.get("/download", (req, res) => {
  const requested = path.resolve(baseDir, req.query.file);

  if (!requested.startsWith(baseDir + path.sep)) {
    return res.status(400).send("Invalid file");
  }

  res.sendFile(requested);
});
```

W prawdziwych aplikacjach preferuj file IDs i authorization checks zamiast surowych filenames.

## Notatka dla mnie

`path.join()` pomaga budować ścieżki, ale samo w sobie nie jest security boundary.
