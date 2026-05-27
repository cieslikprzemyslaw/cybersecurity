# A03:2025 - Software Supply Chain Failures

## Co oznacza ta kategoria OWASP

Software Supply Chain Failures dotyczą ryzyka w elementach, którym aplikacja ufa poza własnym kodem: zależnościach, narzędziach builda, package managerze, workflow CI/CD, obrazach Dockera, skryptach instalacyjnych i artefaktach release.

W projekcie Node.js lub frontendowym łańcuch dostaw obejmuje między innymi:

- `package.json`,
- zależności bezpośrednie i transitive dependencies,
- npm registry,
- npm lifecycle scripts,
- lockfile,
- workflow GitHub Actions,
- narzędzia build/test/package,
- obrazy Dockera,
- SBOM i inventory komponentów,
- sekrety używane podczas builda i deploymentu.

W prostych słowach:

> Problem supply chain oznacza, że aplikacja może stać się ryzykowna przez coś, czemu ufa podczas install, build, test, package albo deploy.

## Dlaczego to ważne

Nowoczesne aplikacje frontendowe i Node.js zwykle używają wielu paczek oraz automatycznych kroków pipeline'u. Ryzyko może wejść do aplikacji przez:

- złośliwą paczkę npm,
- skompromitowaną zależność pośrednią,
- build bez powtarzalnych wersji zależności,
- workflow używający mutable tagów GitHub Actions,
- Docker base image zmieniony poza kontrolą zespołu,
- `curl | sh` w CI,
- sekrety dostępne w jobie, który wykonuje niezaufany kod,
- artefakt release, którego nie da się powiązać z testowanym commitem.

## Kontekst praktycznego review

Dla tej kategorii przejrzałem publiczny projekt Node/JavaScript z perspektywy supply chain. Celem była nauka AppSec review, a nie zgłaszanie podatności do projektu.

Review obejmowało:

- root `package.json`,
- npm scripts i lifecycle scripts,
- `postinstall`,
- `npm install --ignore-scripts`,
- zależności security-sensitive,
- lockfile i powtarzalność buildów,
- `npm install` vs `npm ci`,
- SBOM,
- workflow GitHub Actions,
- pinning akcji,
- wzorce typu `curl | sh`,
- dostęp sekretów w CI/CD.

## Typowe przykłady

Do Software Supply Chain Failures mogą należeć:

- brak lockfile albo ignorowanie lockfile,
- CI używające dynamicznego `npm install`,
- podatne, porzucone lub podszywające się paczki,
- lifecycle scripts uruchamiane automatycznie podczas install,
- remote scripts uruchamiane bez weryfikacji,
- GitHub Actions przypięte tylko do tagów,
- Docker images bez pinning/review,
- brak dependency scanning,
- brak SBOM,
- brak artifact provenance,
- sekrety dostępne w ryzykownych jobach pipeline'u.

## Główne pojęcia

### npm lifecycle scripts

Skrypty takie jak `preinstall`, `install`, `postinstall` i `prepare` mogą wykonać kod automatycznie podczas instalacji paczek. Nie jest to automatycznie podatność, ale jest to ważny punkt review, szczególnie w CI/CD.

### `npm install --ignore-scripts`

`npm install --ignore-scripts` instaluje zależności bez uruchamiania lifecycle scripts. To przydatna kontrola w jobach, które potrzebują paczek do lintingu lub analizy statycznej, ale nie potrzebują uruchamiać kodu instalacyjnego.

### Lockfile i powtarzalność

`package-lock.json`, `yarn.lock` lub `pnpm-lock.yaml` zapisują konkretne wersje zależności. Bez lockfile build dziś i build za miesiąc mogą pobrać inne wersje transitive dependencies mimo braku zmian w kodzie.

### `npm install` vs `npm ci`

`npm install` jest wygodne lokalnie, ale może aktualizować lockfile i rozwiązywać zakresy wersji dynamicznie. `npm ci` jest lepszym domyślnym wyborem dla CI, gdy istnieje lockfile, bo instaluje dokładnie z lockfile i failuje przy niespójności.

### SBOM

SBOM, czyli Software Bill of Materials, to inventory komponentów użytych w aplikacji lub artefakcie. Sam SBOM nie naprawia podatności, ale pomaga w vulnerability management, audytach i incident response.

### GitHub Actions pinning

Akcja użyta jako `@v4` jest wygodna, ale tag może wskazywać na inny kod w przyszłości. Pinning do commit SHA poprawia powtarzalność i zmniejsza ryzyko pobrania zmienionego kodu akcji.

## Powiązane notatki

- [HTTP, Request/Response i podstawy Auth](../../fundamentals/01-http-request-response-auth.md)
- [Content Discovery / Attack Surface](../../fundamentals/03-content-discovery-attack-surface.md)
- [Security Misconfiguration](../a02-security-misconfiguration/01-overview.md)
- [File Upload Vulnerabilities](../../key-web-vulnerabilities/07-file-upload-vulnerabilities/README.md)
- [SSRF Basics](../../key-web-vulnerabilities/08-server-side-request-forgery-ssrf/README.md)

## Wniosek jako Frontend Engineer

Frontend może mieć duży supply chain: paczki npm, bundlery, narzędzia testowe, workflow CI, obrazy i proces deploymentu. AppSec review nie kończy się na request/response. Trzeba też sprawdzić, jak aplikacja jest budowana, co wykonuje się automatycznie i czy finalny artefakt jest powtarzalny oraz możliwy do prześledzenia.

## Co wygląda dobrze

Bezpieczniejszy setup powinien mieć:

- commitowany lockfile,
- `npm ci` w CI tam, gdzie to możliwe,
- świadome lifecycle scripts,
- `--ignore-scripts` w jobach, które nie potrzebują skryptów,
- dependency scanning,
- secret scanning,
- SBOM,
- ograniczone permissions w workflow,
- GitHub Actions przypięte do SHA lub świadomie reviewowane,
- minimalny dostęp do sekretów,
- review remote scripts,
- jasny proces release artifact.

## Podsumowanie

A03 dotyczy zaufania w łańcuchu dostarczania oprogramowania. Najważniejsze punkty z review: lifecycle scripts są istotnym miejscem ryzyka, lockfile i `npm ci` wzmacniają powtarzalność, SBOM daje widoczność, a `curl | sh` i nieprzypięte akcje CI/CD wymagają świadomego review.
