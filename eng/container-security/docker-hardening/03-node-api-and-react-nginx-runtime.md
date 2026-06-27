# Node API and React/nginx Runtime Images

## Purpose

This note explains the runtime image design for both parts of the application:

- Node.js / Express API,
- React / Vite frontend served by nginx.

The main lesson was that backend and frontend need different runtime models.

The API still needs Node.js and production dependencies at runtime.

The frontend does not need Node.js at runtime after Vite creates static files.

---

## API runtime image

The API stack:

```text
Node.js
Express
TypeScript
Prisma
SQLite
local uploads
Zod validation
API routes
health endpoint
```

The API runtime needs:

- compiled JavaScript,
- production `node_modules`,
- generated Prisma client,
- package metadata,
- runtime directories,
- environment variables,
- access to SQLite database path,
- access to uploads path,
- startup command.

It should not need:

- TypeScript source files,
- tests,
- Storybook,
- Playwright,
- lint tools,
- dev dependencies,
- frontend dev server,
- unnecessary local scripts.

Hardening idea:

```text
The API image should be a runtime artifact, not a copy of the developer machine.
```

---

## Why TypeScript source is not runtime

The API is written in TypeScript, but Node runs JavaScript.

Production flow:

```text
TypeScript source
  -> TypeScript build
  -> JavaScript output
  -> Node runtime
```

The runtime command should start compiled output, for example:

```bash
node dist-server/server/index.js
```

This makes the runtime cleaner and more predictable.

Security lesson:

```text
Do not ship source/build tooling unless runtime truly requires it.
```

---

## Prisma in the API image

Prisma requires a generated client.

The build stage needed to run Prisma generation before or during the server build.

Important lesson:

```text
If compiled server code imports generated Prisma code, the generated code must exist in the runtime image in executable form.
```

This became important when the container crashed because generated JavaScript still referenced `.ts` imports.

---

## API runtime problem: Prisma `.ts` import

The API image built successfully, but the API container crashed at runtime.

Error pattern:

```text
Error [ERR_MODULE_NOT_FOUND]:
Cannot find module '/app/dist-server/generated/prisma/internal/class.ts'
imported from /app/dist-server/generated/prisma/client.js
```

Meaning:

```text
Node was running JavaScript.
The generated Prisma client JavaScript still imported a .ts file.
Node could not resolve that TypeScript file at runtime.
```

Fix in `tsconfig.server.json`:

```json
{
  "compilerOptions": {
    "rewriteRelativeImportExtensions": true
  }
}
```

Validation:

```powershell
Get-ChildItem .\dist-server\generated\prisma -Recurse -Filter *.js |
    Select-String -Pattern "\.ts'"
```

Expected:

```text
No problematic .ts import results.
```

Lesson:

```text
Build success does not guarantee runtime success.
```

---

## API production dependency stage

The API runtime needs packages from `node_modules`, but only production packages.

Good production install:

```bash
npm ci --omit=dev --no-audit --no-fund
```

Why not ship all dependencies?

Because dev dependencies may include:

- compilers,
- test frameworks,
- browser automation tools,
- linters,
- Storybook,
- development scripts,
- and packages not needed by the running API.

Security lesson:

```text
Every unnecessary runtime dependency is extra attack surface and extra scan noise.
```

---

## API SQLite persistence

SQLite is not a database server.

It is a database file.

So Docker does not need a separate SQLite container.

It needs persistent file storage.

Runtime config:

```text
DATABASE_URL=file:/data/appsec-report-builder.db
```

Compose volume:

```yaml
volumes:
  - api-data:/data
```

Lesson:

```text
For SQLite, persistence means keeping the database file outside the disposable container filesystem.
```

---

## API upload persistence

The API stores uploaded files.

Uploads are mutable data and should survive container recreation.

Compose volume:

```yaml
volumes:
  - api-uploads:/app/uploads
```

Security note:

```text
Uploaded files are untrusted content stored on disk.
The app needs safe file names, size limits, type validation and path traversal protection.
```

Future cloud version could move uploads to object storage.

---

## API health endpoint

The API health endpoint was:

```text
/api/health
```

Validation:

```powershell
Invoke-WebRequest http://localhost:3000/api/health -UseBasicParsing
```

Expected:

```json
{"status":"ok"}
```

Useful distinction:

```text
connection refused:
  API is not listening or crashed

404 route not found:
  API is running, but endpoint path is wrong

200 OK:
  route works
```

---

## React / Vite frontend runtime

The frontend stack:

```text
React
Vite
TypeScript
SPA routing
API calls
uploads/assets loaded through backend paths
```

In development, Vite provides:

- dev server,
- fast refresh,
- dev proxy,
- source transforms.

In production, Vite produces static files:

```text
index.html
CSS
JavaScript
assets
```

That means the frontend runtime can be nginx.

---

## Frontend runtime does not need Node.js

Build-time:

```text
Node image
npm ci
npm run build:client
```

Runtime:

```text
nginx image
copy dist output
serve static files
```

Key lesson:

```text
React/Vite needs Node to build. It does not need Node to run in production.
```

Security benefit:

- no frontend `node_modules` in runtime,
- no Vite dev server in runtime,
- smaller runtime,
- fewer tools available,
- clearer production behaviour.

---

## nginx for static frontend

nginx was used to:

- serve Vite static output,
- support SPA fallback,
- proxy `/api` to backend,
- proxy `/uploads` to backend.

The runtime used an unprivileged nginx image:

```text
nginxinc/nginx-unprivileged:stable-alpine
```

Security lesson:

```text
Prefer an unprivileged runtime image when the service does not need elevated privileges.
```

---

## SPA fallback

React routes may not exist as real files.

Example:

```text
/settings
/companies
/assessments
```

If nginx looks for those files directly, it may return 404 on refresh.

SPA fallback:

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

Meaning:

```text
try actual file
try directory
otherwise serve index.html
```

React Router then handles the route in the browser.

---

## Development proxy vs production proxy

Vite dev proxy works only during development.

After production build, there is no Vite server.

So production needs a real proxy layer.

nginx handled:

```text
/api     -> http://api:3000/api/
/uploads -> http://api:3000/uploads/
```

Lesson:

```text
Frontend development behaviour is not production behaviour.
```

This is a very practical frontend security/deployment lesson.

---

## nginx proxy flow

Request:

```text
http://localhost:8080/api/health
```

Flow:

```text
Browser
  -> web nginx container
  -> http://api:3000/api/health
  -> API container
```

Inside Docker Compose, `api` is the service name and DNS name.

Do not use `localhost` from the web container to reach the API container.

Inside a container:

```text
localhost = this same container
```

---

## Final runtime split

API runtime:

```text
Node.js runtime
compiled server
production dependencies
SQLite volume
uploads volume
health endpoint
```

Web runtime:

```text
nginx runtime
static frontend files
SPA fallback
/api proxy
/uploads proxy
```

Security lesson:

```text
Different parts of the stack should have different runtime images based on what they actually need.
```

---

## Key takeaway

Backend and frontend Docker images should not be designed the same way.

The API needs a Node runtime.

The frontend needs a static file server.

The practical hardening rule:

```text
Build with the tools you need.
Run with only the tools you must have.
```
