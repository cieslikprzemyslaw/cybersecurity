# Node.js Upload Risks

In Node.js/Express applications, uploading a `.js` file normally does not execute it on the server.

Main risks:

- path traversal through filename
- unsafe static file serving
- stored XSS through HTML/SVG
- file overwrite
- unsafe parsing of uploaded content

## Main Takeaway

In Node.js, unsafe storage, static serving and processing are often more relevant than direct execution.
