# A03:2025 - Software Supply Chain Failures: laby i praktyka

## Cel

Ten plik dokumentuje praktyczne review A03:2025 - Software Supply Chain Failures. Celem było przećwiczenie myślenia supply chain na publicznym projekcie Node/JavaScript, a nie zgłaszanie podatności do tego projektu.

Ta kategoria jest bardziej związana z procesem, buildem, zależnościami i CI/CD niż z klasycznym request/response testing.

## Zakres praktyki

**Typ celu:** publiczny projekt open-source Node/JavaScript  
**Focus:** review paczek i CI/CD  
**Kategoria:** A03:2025 - Software Supply Chain Failures  
**Status:** ukończone po review i quizie

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

W `package.json` skupiłem się na:

- `scripts`,
- `dependencies` i `devDependencies`,
- lifecycle scripts,
- paczkach security-sensitive,
- build/package scripts,
- SBOM.

Najciekawszy skrypt:

```json
"postinstall": "cd frontend && npm install && cd .. && npm run build:frontend && (npm run --silent build:server || cd .)"
```

### Czego nauczyłem się o `postinstall`

`postinstall` uruchamia się automatycznie po `npm install`. Taki skrypt może być uzasadniony w złożonym projekcie, ale z perspektywy supply chain jest ważnym punktem review, bo wykonuje kod podczas instalacji.

W tym przykładzie skrypt:

1. wchodzi do katalogu `frontend`,
2. uruchamia kolejne `npm install`,
3. wraca do root,
4. buduje frontend,
5. próbuje zbudować server.

Pytanie AppSec nie brzmi "czy `postinstall` jest zawsze zły", tylko:

> Czy skrypt wykonuje wyłącznie oczekiwane, przejrzane komendy i czy jest bezpieczny lokalnie oraz w CI/CD?

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

## Czego się nauczyłem

1. Supply chain review to nie tylko CVE w zależnościach.
2. `--ignore-scripts` jest praktyczną kontrolą dla jobów, które nie potrzebują lifecycle scripts.
3. Brak lockfile i dynamiczny `npm install` osłabiają powtarzalność buildów.
4. SBOM jest inventory komponentów, nie automatyczną poprawką.
5. Pinning GitHub Actions do SHA ma znaczenie security i reproducibility.
6. `curl | sh` w CI/CD wymaga bardzo uważnego review.

## Real-world review angle

W realnym projekcie sprawdziłbym:

- czy lockfile jest commitowany,
- czy CI używa `npm ci`,
- gdzie lifecycle scripts są uruchamiane,
- czy można użyć `--ignore-scripts`,
- czy akcje GitHub są przypięte do SHA,
- czy sekrety są dostępne w jobach uruchamiających zależności,
- czy działa dependency scanning i secret scanning,
- czy SBOM jest generowany,
- czy Docker base images są reviewowane,
- czy artefakty release są powiązane z testowanym commitem.

## Powiązane notatki

- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Security Misconfiguration](../a02-security-misconfiguration/01-overview.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)

## Podsumowanie

Najważniejszy wniosek: A03 wymaga patrzenia na cały proces dostarczania aplikacji. Bez powtarzalnego install/build i bez review CI/CD projekt może mieć ryzyko nawet wtedy, gdy sam kod aplikacji wygląda poprawnie.
