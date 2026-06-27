# Container Mental Model

## Why this matters

Docker becomes much easier when the basic objects are clear.

The most important mistake is treating Docker as magic. It is not magic. It is a set of build, filesystem, process, networking and isolation concepts.

For AppSec, the mental model matters because container security depends on knowing:

- what is inside the image,
- what changes at runtime,
- what persists,
- what is exposed,
- and what the process is allowed to do.

---

## Image

A Docker image is a read-only template used to create containers.

It usually contains:

- base operating system files,
- application files,
- installed dependencies,
- configuration files,
- startup command metadata,
- exposed port metadata,
- and filesystem layers created during build.

An image does not run by itself. A container runs from an image.

Useful command:

```powershell
docker image ls
```

Security question:

```text
What exactly is inside this image, and does the runtime really need it?
```

If the image contains secrets, dev tools, unnecessary packages or source files, they may become available to the running process.

---

## Container

A container is a running instance of an image.

It is a process with:

- a filesystem,
- environment variables,
- network configuration,
- mounted volumes,
- process user,
- and runtime settings.

Useful command:

```powershell
docker ps
```

A container can be stopped, removed and recreated.

That is why containers should be treated as disposable. Important data should not live only inside the container filesystem.

Security lesson:

```text
Images contain application code.
Volumes or external services contain mutable data.
```

---

## Image vs container

| Concept | Meaning |
|---|---|
| Image | Build artifact / template |
| Container | Running process created from image |
| Dockerfile | Instructions for building an image |
| Docker Compose | Instructions for running services together |
| Volume | Persistent storage outside container lifecycle |
| Network | Communication layer between containers |

Practical difference:

```text
docker build creates an image
docker run creates a container from an image
docker compose up creates multiple containers/services
```

---

## Layer

A Docker image is built from layers.

Each Dockerfile instruction can create a layer.

Example:

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build
```

Layers matter because:

- Docker can cache them,
- changing an earlier layer invalidates later layers,
- secrets copied into a layer can be hard to remove safely,
- large copied directories make images bigger,
- and layer order affects build speed.

Security lesson:

```text
Do not copy secrets into an image and delete them later.
Use build secrets instead.
```

---

## Build context

The build context is the set of files sent to Docker during build.

Example:

```powershell
docker build -f docker/api/Dockerfile .
```

The final `.` is the build context.

That means Docker can access files under the current directory unless excluded by `.dockerignore`.

Why it matters:

- large context makes builds slower,
- accidental files may be available during build,
- secrets may be copied accidentally,
- `COPY . .` may include more than expected.

Typical `.dockerignore` entries:

```text
node_modules
dist
dist-server
.git
.env
*.log
coverage
uploads
*.db
```

Security lesson:

```text
Reduce the build context. Do not send unnecessary local files to Docker.
```

---

## Base image

A base image is the starting point in a `FROM` instruction.

Examples:

```dockerfile
FROM node:24-bookworm
```

```dockerfile
FROM nginxinc/nginx-unprivileged:stable-alpine
```

Questions to ask:

- Is this image trusted?
- Is it maintained?
- Is it larger than necessary?
- Does it run as root?
- Does it contain tools that runtime does not need?

A build stage can use a larger image because it needs build tools.

A runtime stage should usually be smaller and cleaner.

---

## Dockerfile

A Dockerfile answers:

```text
How do I build this image?
```

It describes files, commands, dependencies and startup metadata for one image.

Common instructions:

```dockerfile
FROM
WORKDIR
COPY
RUN
ENV
USER
EXPOSE
CMD
HEALTHCHECK
```

Security lesson:

```text
The Dockerfile controls what becomes part of the final runtime environment.
Bad Dockerfile decisions can ship secrets, dev tools or unnecessary dependencies.
```

---

## Docker Compose

Docker Compose answers:

```text
How do these services run together?
```

It can define:

- services,
- images/builds,
- environment variables,
- ports,
- volumes,
- dependencies,
- networks,
- restart policies.

In the lab, Compose ran:

```text
api-migrate
api
web
```

Security lesson:

```text
A clean image can still be run insecurely if the Compose/runtime configuration is weak.
```

---

## Volume

A Docker volume is persistent storage managed by Docker.

It exists outside the lifecycle of one container.

In the lab:

```text
api-data     -> SQLite database file
api-uploads  -> uploaded files
```

This matters because containers are disposable.

Without volumes, deleting/recreating a container can delete important data.

Security lesson:

```text
Mutable data should be explicit. Do not rely on accidental writes inside the container filesystem.
```

---

## Network

Docker Compose creates a default network for services.

Services can talk to each other by service name.

Inside Docker network:

```text
web -> http://api:3000
```

From the host machine:

```text
browser -> http://localhost:8080
browser -> http://localhost:3000
```

Important distinction:

```text
Inside a container, localhost means the same container.
It does not mean the host and it does not mean another service.
```

Security lesson:

```text
Only publish ports that need host access. Internal service communication can stay inside Docker networks.
```

---

## Build-time vs runtime

Build-time tasks:

- install dependencies,
- compile TypeScript,
- run Vite build,
- generate Prisma client,
- create build artifacts.

Runtime tasks:

- start Node API,
- serve static files with nginx,
- read environment variables,
- connect to database,
- write uploaded files,
- respond to health checks.

Security lesson:

```text
Build-time tools should not automatically exist at runtime.
If the app does not need a tool to run, keep it out of the final image.
```

---

## Key takeaway

A clean Docker mental model has four layers:

```text
Build:
  Dockerfile creates image

Image:
  read-only runtime template

Run:
  container starts from image

State:
  volumes/external services persist data
```

For AppSec, each layer has different risks:

```text
Build risks:
  leaking secrets, unsafe dependency installation, untrusted scripts

Image risks:
  too many tools, vulnerable packages, source code leakage

Runtime risks:
  root user, writable filesystem, excessive capabilities, exposed ports

State risks:
  lost data, weak permissions, untrusted uploads, secrets stored in files
```
