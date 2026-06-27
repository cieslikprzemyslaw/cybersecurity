# Runtime Images dla Node API i React/nginx

## Cel

Ta notatka wyjaśnia projekt runtime image dla dwóch części aplikacji:

- Node.js / Express API,
- frontend React / Vite serwowany przez nginx.

Najważniejsza lekcja: backend i frontend potrzebują różnych modeli runtime.

API nadal potrzebuje Node.js i produkcyjnych zależności w runtime.

Frontend po buildzie Vite nie potrzebuje Node.js w runtime.

---

## Runtime image API

Stack API:

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

API runtime potrzebuje:

- skompilowanego JavaScript,
- production `node_modules`,
- wygenerowanego klienta Prisma,
- package metadata,
- katalogów runtime,
- environment variables,
- dostępu do ścieżki bazy SQLite,
- dostępu do ścieżki uploadów,
- komendy startowej.

Nie powinien potrzebować:

- plików źródłowych TypeScript,
- testów,
- Storybooka,
- Playwrighta,
- narzędzi lintowania,
- dev dependencies,
- frontend dev servera,
- zbędnych lokalnych skryptów.

```text
Obraz API powinien być artefaktem runtime, nie kopią maszyny developerskiej.
```

---

## Dlaczego TypeScript source nie jest runtime

API jest napisane w TypeScript, ale Node uruchamia JavaScript.

```text
TypeScript source
  -> TypeScript build
  -> JavaScript output
  -> Node runtime
```

Komenda runtime powinna uruchamiać skompilowany output:

```bash
node dist-server/server/index.js
```

```text
Nie wysyłaj source/build tooling, jeżeli runtime tego nie wymaga.
```

---

## Prisma w obrazie API

Prisma wymaga wygenerowanego klienta.

Build stage musiał uruchomić generowanie Prisma przed lub podczas builda serwera.

```text
Jeżeli skompilowany server code importuje wygenerowany kod Prisma,
ten kod musi istnieć w runtime image w wykonywalnej formie.
```

To było ważne, gdy kontener padał, bo wygenerowany JavaScript nadal odwoływał się do importów `.ts`.

---

## Problem runtime: import `.ts` z Prisma

Obraz API zbudował się poprawnie, ale kontener API padał w runtime.

Wzorzec błędu:

```text
Error [ERR_MODULE_NOT_FOUND]:
Cannot find module '/app/dist-server/generated/prisma/internal/class.ts'
imported from /app/dist-server/generated/prisma/client.js
```

Znaczenie:

```text
Node uruchamiał JavaScript.
Wygenerowany klient Prisma w JavaScript nadal importował plik .ts.
Node nie mógł rozwiązać tej ścieżki w runtime.
```

Poprawka w `tsconfig.server.json`:

```json
{
  "compilerOptions": {
    "rewriteRelativeImportExtensions": true
  }
}
```

Walidacja:

```powershell
Get-ChildItem .\dist-server\generated\prisma -Recurse -Filter *.js |
    Select-String -Pattern "\.ts'"
```

Oczekiwany wynik:

```text
Brak problematycznych importów .ts.
```

---

## Production dependency stage

API runtime potrzebuje pakietów z `node_modules`, ale tylko produkcyjnych.

```bash
npm ci --omit=dev --no-audit --no-fund
```

Dlaczego nie wysyłać wszystkich dependencies?

Dev dependencies mogą zawierać:

- kompilatory,
- test frameworks,
- browser automation tools,
- lintery,
- Storybook,
- skrypty development,
- pakiety niepotrzebne działającemu API.

```text
Każda zbędna zależność runtime to dodatkowa powierzchnia ataku i szum w skanach podatności.
```

---

## Trwałość SQLite

SQLite nie jest osobnym serwerem bazy.

To plik bazy danych.

Docker nie potrzebuje osobnego kontenera SQLite. Potrzebuje trwałego storage na plik.

```text
DATABASE_URL=file:/data/appsec-report-builder.db
```

```yaml
volumes:
  - api-data:/data
```

```text
Dla SQLite w Dockerze persistence oznacza trwałość pliku poza disposable container filesystem.
```

---

## Trwałość uploadów

API zapisuje uploadowane pliki.

Uploady są mutable data i powinny przetrwać odtworzenie kontenera.

```yaml
volumes:
  - api-uploads:/app/uploads
```

```text
Uploadowane pliki są niezaufaną treścią zapisaną na dysku.
Aplikacja potrzebuje bezpiecznych nazw, limitów rozmiaru, walidacji typu i ochrony przed path traversal.
```

W wersji cloud uploady mogłyby trafić do object storage.

---

## Health endpoint API

Endpoint:

```text
/api/health
```

Walidacja:

```powershell
Invoke-WebRequest http://localhost:3000/api/health -UseBasicParsing
```

Oczekiwane:

```json
{"status":"ok"}
```

Przydatne rozróżnienie:

```text
connection refused:
  API nie nasłuchuje albo padło

404 route not found:
  API działa, ale ścieżka endpointu jest błędna

200 OK:
  route działa
```

---

## Runtime frontendu React / Vite

Stack frontendu:

```text
React
Vite
TypeScript
SPA routing
API calls
uploads/assets ładowane przez backend paths
```

W development Vite daje:

- dev server,
- fast refresh,
- dev proxy,
- transformacje source.

W production Vite produkuje statyczne pliki:

```text
index.html
CSS
JavaScript
assets
```

Runtime frontendu może więc być nginx.

---

## Frontend runtime bez Node.js

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

```text
React/Vite potrzebuje Node do builda. W production runtime Node nie jest potrzebny.
```

Korzyści:

- brak frontend `node_modules` w runtime,
- brak Vite dev servera w runtime,
- mniejszy runtime,
- mniej narzędzi,
- czytelniejsze zachowanie produkcyjne.

---

## nginx dla statycznego frontendu

nginx służył do:

- serwowania statycznego outputu Vite,
- SPA fallback,
- proxy `/api` do backendu,
- proxy `/uploads` do backendu.

Runtime używał unprivileged nginx image:

```text
nginxinc/nginx-unprivileged:stable-alpine
```

```text
Preferuj unprivileged runtime image, jeżeli usługa nie potrzebuje podwyższonych uprawnień.
```

---

## SPA fallback

Trasy React nie muszą istnieć jako realne pliki.

```text
/settings
/companies
/assessments
```

Jeżeli nginx będzie szukał tych plików bezpośrednio, refresh strony może zwrócić 404.

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

Znaczenie:

```text
spróbuj realnego pliku
spróbuj katalogu
w przeciwnym razie zwróć index.html
```

React Router obsłuży trasę w przeglądarce.

---

## Development proxy vs production proxy

Vite dev proxy działa tylko w development.

Po production build nie ma serwera Vite.

Production potrzebuje prawdziwej warstwy proxy.

nginx obsługiwał:

```text
/api     -> http://api:3000/api/
/uploads -> http://api:3000/uploads/
```

```text
Zachowanie frontendu w development nie jest zachowaniem produkcyjnym.
```

---

## Przepływ proxy nginx

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

W Docker Compose `api` jest nazwą usługi i nazwą DNS.

Nie używaj `localhost` z kontenera web, żeby dotrzeć do API.

```text
localhost = ten sam kontener
```

---

## Finalny podział runtime

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

```text
Różne części stacka powinny mieć różne runtime images zależnie od tego, czego naprawdę potrzebują.
```

---

## Najważniejszy wniosek

Backend i frontend nie powinny mieć projektowanych obrazów Dockera w ten sam sposób.

API potrzebuje Node runtime.

Frontend potrzebuje static file server.

```text
Buduj z narzędziami, których potrzebujesz.
Uruchamiaj tylko z narzędziami, które są naprawdę wymagane.
```
