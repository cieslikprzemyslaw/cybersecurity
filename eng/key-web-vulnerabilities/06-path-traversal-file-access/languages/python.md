# Python - Path Traversal

## Risky APIs and patterns

| API / pattern | Risk |
|---|---|
| `open()` | Arbitrary file read |
| `send_file()` | File disclosure in Flask if path is user-controlled |
| `FileResponse` | File disclosure in Django/FastAPI if path is user-controlled |
| `os.path.join()` | Not enough by itself |
| template/file loading from input | Can lead to file access bugs |

## Dangerous pattern

```python
@app.route("/download")
def download():
    filename = request.args.get("file")
    return send_file("/app/files/" + filename)
```

## Safer approach

Use IDs or allowlisted filenames where possible.

If path handling is required, resolve the final path and check it stays inside the base directory.

```python
from pathlib import Path

base = Path("/app/files").resolve()
requested = (base / filename).resolve()

if not str(requested).startswith(str(base) + "/"):
    abort(400)
```

## Note to self

The risky moment is passing request data into `open()`, `send_file()` or similar helpers.
