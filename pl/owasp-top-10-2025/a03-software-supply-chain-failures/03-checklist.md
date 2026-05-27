# A03:2025 - Software Supply Chain Failures checklist

## Cel

Użyj tej checklisty przy review projektu frontendowego, Node.js albo full-stack JavaScript pod kątem ryzyk supply chain.

Główne pytanie:

> Czy coś poza naszym kodem źródłowym może zmienić to, co zostanie zainstalowane, zbudowane, przetestowane, spakowane albo wdrożone?

## 1. Review `package.json`

- Jakie skrypty uruchamiają install, build, test, package i deploy?
- Czy istnieją `preinstall`, `install`, `postinstall` albo `prepare`?
- Czy skrypty pobierają pliki z internetu?
- Czy skrypty wykonują remote code?
- Czy skrypty ukrywają błędy przez `|| true` lub podobny wzorzec?
- Czy uruchamiają nested `npm install`?
- Czy mają dostęp do sekretów albo environment variables?

Red flags:

- nieoczekiwany `postinstall`,
- `curl ... | sh`,
- `wget ... | bash`,
- dynamic downloads during build,
- nested install bez jasnego powodu,
- skrypty instalacyjne w jobach z sekretami.

## 2. Lifecycle scripts

Sprawdź:

```text
preinstall
install
postinstall
prepare
prepublish
prepack
postpack
```

Pytania:

- Czy skrypt musi działać w CI?
- Czy wykonuje third-party code?
- Czy pobiera coś z sieci?
- Czy kompiluje native modules?
- Czy potrzebuje sekretów?
- Czy w części jobów można użyć `--ignore-scripts`?

## 3. Dependencies

- Które zależności są bezpośrednie?
- Które są transitive?
- Czy są przestarzałe, porzucone albo podatne?
- Czy nazwy paczek wyglądają podejrzanie?
- Czy dependency powinno być w `devDependencies` zamiast `dependencies`?
- Czy update'y są automatyzowane i reviewowane?

Szczególnie reviewuj paczki od:

- auth/JWT/session/cookies,
- sanitizacji,
- uploadu plików,
- archive extraction,
- XML/YAML parsing,
- template rendering,
- CORS,
- security headers,
- error handling,
- native/binary install,
- HTTP clients.

## 4. Lockfile i powtarzalność

- Czy lockfile jest commitowany?
- Czy projekt używa `package-lock.json`, `yarn.lock` albo `pnpm-lock.yaml`?
- Czy CI używa `npm ci` tam, gdzie może?
- Czy `npm install` w CI ma uzasadnienie?
- Czy package manager config nie wyłącza lockfile?
- Czy zmiany lockfile są reviewowane w PR?

Red flags:

- brak lockfile,
- lockfile w `.gitignore`,
- `package-lock=false`,
- wiele kroków `npm install`,
- broad version ranges bez kontroli lockfile.

## 5. CI/CD dependency install

- Które joby instalują zależności?
- Czy używają `npm install`, `npm ci` czy innego package managera?
- Czy install scripts się uruchamiają?
- Czy job ma sekrety w czasie instalacji?
- Czy global installs są przypięte do wersji?
- Czy dependency install dzieje się przed czy po udostępnieniu sekretów?

## 6. `--ignore-scripts`

`npm install --ignore-scripts` instaluje paczki bez lifecycle scripts.

Sprawdź:

- czy lint/static analysis joby mogą go użyć,
- gdzie lifecycle scripts są naprawdę potrzebne,
- czy różnica między jobami jest udokumentowana,
- czy kontrola nie daje fałszywego poczucia bezpieczeństwa w jobach, które nadal używają zwykłego `npm install`.

## 7. SBOM

- Czy SBOM jest generowany?
- Czy powstaje w CI/release, a nie tylko lokalnie?
- Czy format to CycloneDX lub SPDX?
- Czy SBOM jest dołączany do artefaktu?
- Czy jest używany w vulnerability management?
- Czy obejmuje production dependencies?

## 8. Dependency scanning i updates

- Czy działa Dependabot, Renovate, Snyk, OSV Scanner, Trivy, Grype albo podobne narzędzie?
- Czy alerty mają właściciela?
- Czy high/critical findings są triage'owane?
- Czy devDependency findings są oceniane osobno od runtime findings?
- Czy security updates są testowane przed merge?

## 9. GitHub Actions

- Czy actions są pinned to commit SHA?
- Czy są tylko mutable tagi typu `@v4`?
- Czy third-party actions są zaufane i reviewowane?
- Czy permissions są minimalne?
- Czy sekrety nie trafiają do untrusted PR workflows?
- Czy publish/deploy działa tylko z trusted branches?
- Czy `pull_request_target` nie wykonuje niezaufanego kodu?

## 10. Remote script execution

Reviewuj wzorce:

```bash
curl https://example.com/install.sh | sh
wget https://example.com/install.sh -O- | bash
```

Sprawdź:

- czy wersja instalatora jest przypięta,
- czy jest checksum lub signature verification,
- czy job ma sekrety,
- czy można użyć trusted package managera,
- czy endpoint jest zaufany i stabilny.

## 11. Docker i artefakty

- Czy Docker base images są reviewowane albo pinned by digest?
- Czy image scanning działa?
- Czy final image jest minimalny?
- Czy działa non-root user tam, gdzie to możliwe?
- Czy SBOM jest generowany dla image?
- Czy image jest podpisany lub ma attestation?
- Czy deployowany artefakt jest tym, który przeszedł testy?

## 12. Secrets w CI/CD

- Które joby mają dostęp do sekretów?
- Czy te joby uruchamiają dependency scripts?
- Czy sekrety są dostępne w PR z forków?
- Czy tokeny mają minimalny zakres?
- Czy secret scanning działa?
- Czy sekrety są rotowane po ekspozycji?

## Remediation checklist

Dla braku powtarzalności:

- dodaj i commituj lockfile,
- użyj `npm ci` w CI,
- reviewuj lockfile diffs,
- skanuj zależności z lockfile.

Dla lifecycle script risk:

- reviewuj i dokumentuj lifecycle scripts,
- używaj `--ignore-scripts` tam, gdzie można,
- ogranicz sekrety w jobach instalacyjnych,
- unikaj network downloads podczas install.

Dla CI/CD risk:

- pin actions do SHA,
- ogranicz workflow permissions,
- reviewuj third-party actions,
- unikaj `curl | sh` albo weryfikuj installer,
- generuj SBOM i utrzymuj dependency scanning.

## Wniosek frontendowy

Bezpieczny frontend to nie tylko brak XSS w komponentach. Trzeba też reviewować paczki, skrypty, lockfile, workflow, obrazy, artefakty i sekrety w pipeline.
