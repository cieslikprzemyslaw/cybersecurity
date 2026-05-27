# A03:2025 - Software Supply Chain Failures: laby i praktyka

## Cel

Ten plik dokumentuje praktyczne review A03:2025 - Software Supply Chain Failures. Celem było przećwiczenie myślenia supply chain na publicznym projekcie Node/JavaScript, a nie zgłaszanie podatności do tego projektu.

Ta kategoria jest bardziej związana z procesem, buildem, zależnościami i CI/CD niż z klasycznym request/response testing.

## Ukończona praktyka

### OWASP Juice Shop - review repozytorium GitHub

**Cel:** OWASP Juice Shop
**Źródło:** review repozytorium GitHub
**Typ celu:** publiczny projekt open-source Node/JavaScript  
**Focus:** review paczek i CI/CD  
**Kategoria:** A03:2025 - Software Supply Chain Failures  
**Status:** ukończone po review i quizie
**Główny pattern:** review supply chain dla npm lifecycle scripts, powtarzalności zależności, SBOM i kontroli GitHub Actions

Link zewnętrzny:

- https://github.com/juice-shop/juice-shop

Sprawdzone obszary:

- root `package.json`,
- npm scripts,
- lifecycle scripts,
- `postinstall`,
- kategorie zależności,
- SBOM,
- workflow CI/CD,
- `npm install` vs `npm ci`,
- `--ignore-scripts`,
- GitHub Actions pinning,
- wzorzec remote script execution,
- lockfile i powtarzalność.

## Praktyka 1: review `package.json`

### Co sprawdziłem

W `package.json` skupiłem się na punktach review supply chain:

- `scripts`,
- `dependencies` i `devDependencies`,
- lifecycle scripts,
- paczkach security-sensitive,
- build/package scripts,
- SBOM.

Głównym punktem review był skrypt `postinstall`:

```json
"postinstall": "cd frontend && npm install && cd .. && npm run build:frontend && (npm run --silent build:server || cd .)"
```

Nie jest to automatycznie podatność, ale jest to ważny punkt review, bo npm lifecycle scripts mogą wykonywać kod podczas instalacji.

### Zależności warte dodatkowego review

Zwróciłem uwagę na paczki obsługujące obszary security-sensitive:

```text
jsonwebtoken / express-jwt     -> tokeny i auth
sanitize-html                  -> sanitizacja HTML / XSS
multer                         -> upload plików
unzipper                       -> rozpakowywanie archiwów
libxmljs2                      -> XML parsing
js-yaml                        -> YAML parsing
cors                           -> CORS
serve-index                    -> directory listing
errorhandler                   -> development-style errors
helmet                         -> security headers
node-pre-gyp / sqlite3         -> native/binary install surface
```

To nie oznacza, że te paczki są podatne. Oznacza, że ich użycie i konfiguracja powinny być reviewowane.

## Praktyka 2: review CI/CD

### Co sprawdziłem

W workflow GitHub Actions sprawdziłem:

- instalację Node.js,
- dependency install,
- linting,
- testy,
- packaging,
- Docker build/push,
- deployment,
- SBOM,
- pinning akcji,
- dostęp sekretów.

### Pozytywna kontrola: `npm install --ignore-scripts`

W jobie lint widoczny był wzorzec:

```bash
npm install --ignore-scripts
cd frontend
npm install --ignore-scripts
```

To ogranicza ryzyko automatycznego wykonania lifecycle scripts takich jak:

```text
preinstall
install
postinstall
prepare
```

Kontrola działa tylko w jobach, w których jest użyta. Jeśli inne joby uruchamiają normalne `npm install`, ryzyko lifecycle scripts nadal trzeba reviewować.

### Najsilniejszy punkt review: powtarzalność

Najważniejszy problem dotyczył braku wyraźnie używanego, commitowanego lockfile przy wielu krokach `npm install`. Bez lockfile dependency tree może zmienić się bez zmiany kodu źródłowego.

Silniejszy wzorzec dla CI to:

- commitowany lockfile,
- `npm ci`,
- review zmian w lockfile,
- dependency scanning.

### SBOM

Projekt miał konfigurację CycloneDX do generowania SBOM. SBOM pomaga odpowiedzieć na pytania:

- jakie komponenty są w artefakcie,
- czy używamy paczki dotkniętej nową podatnością,
- co trzeba sprawdzić podczas incydentu,
- co faktycznie znalazło się w release.

SBOM daje widoczność, ale nie zastępuje skanowania i procesu remediation.

### GitHub Actions i `curl | sh`

Pozytywną kontrolą było przypięcie wielu akcji do commit SHA. Review wymagały akcje użyte tylko przez tagi oraz wzorzec:

```bash
curl https://cli-assets.heroku.com/install.sh | sh
```

Taki wzorzec pobiera i od razu wykonuje zdalny skrypt, więc trzeba sprawdzić wersjonowanie, checksum/signature verification, uprawnienia joba i dostęp do sekretów.

## Najważniejsze wnioski

Najważniejsze lekcje A03 z tej praktyki:

1. `postinstall` i lifecycle scripts są ważnym punktem review supply chain.
2. `--ignore-scripts` może ograniczać install-time code execution, gdy jest użyte świadomie.
3. Brak lockfiles może osłabiać powtarzalność buildów.
4. `npm ci` jest zwykle mocniejsze niż `npm install` w CI, jeśli istnieje lockfile.
5. SBOM daje widoczność dependencies, ale sam nie naprawia podatności.
6. `curl | sh` wymaga uważnego review w CI/CD.
7. SHA-pinned GitHub Actions poprawiają powtarzalność i zmniejszają ryzyko third-party actions.

Dłuższe refleksje i real-world review angle są w:

- [A03 learning notes](05-learning-notes.md)
- [A03 overview](01-overview.md)
- [A03 checklista](03-checklist.md)
- [A03 testy regresji](04-regression-tests.md)
- [Przykładowe znalezisko supply chain](security-findings/01-example-finding.md)

## Powiązane notatki

- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Security Misconfiguration](../a02-security-misconfiguration/01-overview.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)
