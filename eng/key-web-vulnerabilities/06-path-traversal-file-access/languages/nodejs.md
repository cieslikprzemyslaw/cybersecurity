# Node.js - Path Traversal

## Risky APIs

| API | Risk |
|---|---|
| `fs.readFile()` | Arbitrary file read |
| `fs.createReadStream()` | Arbitrary file streaming |
| `res.sendFile()` | File disclosure if path is user-controlled |
| `path.join()` | Useful but not enough by itself |
| `path.resolve()` | Useful only with base directory validation |
| static file middleware | Misconfiguration can expose files |

## Dangerous pattern

```js
app.get("/download", (req, res) => {
  res.sendFile("/app/files/" + req.query.file);
});
```

## Safer approach

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

In real applications, prefer file IDs and authorization checks rather than raw filenames.

## Note to self

`path.join()` is useful for path building, but it is not a security boundary by itself.
