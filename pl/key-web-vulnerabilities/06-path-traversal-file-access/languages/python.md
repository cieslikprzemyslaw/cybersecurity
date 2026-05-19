# Python - Path Traversal

## Ryzykowne API i wzorce

| API / pattern | Ryzyko |
|---|---|
| `open()` | Arbitrary file read |
| `send_file()` | File disclosure we Flask, jeśli path jest user-controlled |
| `FileResponse` | File disclosure w Django/FastAPI, jeśli path jest user-controlled |
| `os.path.join()` | Samo nie wystarcza |
| template/file loading z inputu | Może prowadzić do file access bugs |

## Niebezpieczny wzorzec

```python
@app.route("/download")
def download():
    filename = request.args.get("file")
    return send_file("/app/files/" + filename)
```

## Bezpieczniejsze podejście

Gdzie możliwe, używaj IDs albo allowlisted filenames.

Jeśli obsługa ścieżek jest konieczna, resolve final path i sprawdź, czy zostaje wewnątrz base directory.

```python
from pathlib import Path

base = Path("/app/files").resolve()
requested = (base / filename).resolve()

if not str(requested).startswith(str(base) + "/"):
    abort(400)
```

## Notatka dla mnie

Ryzykowny moment to przekazanie danych z requestu do `open()`, `send_file()` albo podobnych helperów.
