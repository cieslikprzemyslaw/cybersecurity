# A03:2025 - Software Supply Chain Failures testy regresji

## Cel

Testy regresji dla A03 powinny pilnować, żeby kontrole dependency, build, CI/CD i release nie osłabły po czasie.

Nie zawsze są to klasyczne unit testy. Często będą to policy checks, CI checks, dependency scanning, repository rules i manualna weryfikacja release.

## Obszar 1: lockfile

Pozytywny check:

```text
package.json exists
package-lock.json exists
```

albo odpowiednik package managera:

```text
yarn.lock
pnpm-lock.yaml
```

Negatywny check:

```text
package.json exists but no approved lockfile exists.
```

Build powinien failować albo wymagać jawnego wyjątku.

## Obszar 2: `npm ci` w CI/CD

Gdy lockfile istnieje, CI powinno preferować:

```bash
npm ci
```

zamiast:

```bash
npm install
```

Oczekiwane zachowanie:

```text
npm ci succeeds when package.json and package-lock.json are in sync.
npm ci fails when they are out of sync.
```

## Obszar 3: lifecycle script control

Joby, które nie potrzebują lifecycle scripts, powinny używać:

```bash
npm install --ignore-scripts
```

Reviewuj szczególnie:

```text
preinstall
install
postinstall
prepare
```

PR dodający albo zmieniający taki skrypt powinien wymagać review pod kątem remote downloads, sekretów, modyfikacji artefaktów i ukrywania błędów.

## Obszar 4: dependency scanning

Pozytywny check:

```text
Dependabot alerts
npm audit
OSV Scanner
Snyk
Trivy
Grype
GitHub dependency review
```

High i critical findings powinny mieć właściciela, decyzję i status remediation.

## Obszar 5: SBOM

Release albo package process powinien generować SBOM, na przykład:

```bash
cyclonedx-npm --omit=dev --output-format=JSON --output-file=bom.json
```

Negatywny check: release failuje albo ostrzega, jeśli wymagany SBOM nie powstał.

## Obszar 6: GitHub Actions pinning

Pozytywny wzorzec:

```yaml
uses: actions/checkout@<commit-sha>
```

Wzorzec do flagowania:

```yaml
uses: some/action@v1
```

Tag-based actions powinny mieć uzasadnienie albo zostać przypięte do SHA.

## Obszar 7: workflow permissions

Workflow powinien używać minimalnych permissions:

```yaml
permissions:
  contents: read
```

Joby publikujące artefakty, obrazy Dockera albo deployujące aplikację powinny mieć tylko wymagane uprawnienia.

## Obszar 8: secrets exposure

Negatywny check: sekrety nie powinny być dostępne tam, gdzie wykonuje się niezaufany kod albo dependency scripts.

Ryzykowne połączenia:

```text
npm install runs lifecycle scripts
+
job has deployment token
```

```text
pull_request_target
+
checkout and execution of untrusted PR code
```

## Obszar 9: remote script execution

Flaguj:

```bash
curl https://example.com/install.sh | sh
wget https://example.com/install.sh -O- | bash
```

Manualnie zweryfikuj:

- dlaczego skrypt jest potrzebny,
- czy wersja jest przypięta,
- czy jest checksum/signature verification,
- czy job ma sekrety,
- czy istnieje bezpieczniejsza metoda instalacji.

## Obszar 10: Docker i release artifact

Sprawdź:

- czy base images są reviewowane albo pinned by digest,
- czy image scanning działa,
- czy final image jest minimalny,
- czy image działa jako non-root tam, gdzie to możliwe,
- czy SBOM powstaje dla image,
- czy artefakt wdrożony jest powiązany z testowanym commitem.

## Manual verification checklist

Przed zamknięciem A03 review sprawdź:

- lockfiles istnieją i są commitowane,
- CI używa `npm ci` tam, gdzie można,
- lifecycle scripts są reviewowane,
- `--ignore-scripts` jest użyte tam, gdzie pasuje,
- dependency scanning działa,
- SBOM jest generowany,
- GitHub Actions są pinned albo reviewowane,
- workflow permissions są minimalne,
- sekrety są scope'owane ostrożnie,
- `curl | sh` jest unikane albo zweryfikowane,
- release artifacts są traceable.

## Co powinno failować po poprawce

```text
CI install without lockfile.
CI package job using dynamic dependency resolution without justification.
New postinstall script without review.
Remote curl | sh script without verification.
Third-party GitHub Action added without pinning or review.
Release without required SBOM generation.
Dependency scan with untriaged critical finding.
```

## Co powinno nadal działać

```text
Local development install flow.
CI lint/test/build workflows.
Frontend and backend builds.
SBOM generation.
Dependency update process.
Release packaging.
Docker build and smoke tests.
```

## Acceptance criteria

Remediation można zaakceptować, gdy:

- dependency install jest powtarzalny,
- CI używa lockfile-aware installation,
- lifecycle scripts są kontrolowane,
- dependency scanning jest aktywne,
- SBOM jest częścią release/package flow,
- actions i remote scripts są reviewowane,
- sekrety są właściwie scope'owane,
- release artifacts są traceable do kodu i testów.

## Podsumowanie

Główny cel A03 regression checks:

> dependency installation, CI/CD execution i release artifacts powinny być predictable, reviewable i auditable.
