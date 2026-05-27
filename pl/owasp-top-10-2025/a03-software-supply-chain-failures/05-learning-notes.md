# A03:2025 - Software Supply Chain Failures: learning notes

Te notatki zachowują dłuższe refleksje z review repozytorium GitHub dla A03. Krótszy zapis praktyki znajduje się w [02-labs-or-practice.md](02-labs-or-practice.md).

## `postinstall`

`postinstall` uruchamia się automatycznie po `npm install`.

Przed tym review wiedziałem, że npm scripts istnieją, ale nie patrzyłem wcześniej na `postinstall` tak mocno z perspektywy supply chain.

Nauczyłem się, że taki skrypt jest ważny, bo może wykonywać komendy automatycznie podczas instalacji.

W reviewowanym projekcie skrypt:

1. wchodzi do katalogu `frontend`,
2. uruchamia kolejne `npm install`,
3. wraca do root,
4. buduje frontend,
5. próbuje zbudować server.

To może być uzasadnione w złożonym projekcie. Może pomagać automatycznie przygotować zagnieżdżone aplikacje.

Z perspektywy supply chain jest to jednak punkt review, bo lifecycle scripts mogą wykonywać kod podczas instalacji. Jeśli paczka albo dependency zostanie podmienione lub przejęte, install-time scripts są jednym z miejsc, gdzie może pojawić się wykonanie kodu.

Ważny niuans:

> `postinstall` nie jest automatycznie podatnością.

Pytanie AppSec brzmi:

> Czy skrypt wykonuje wyłącznie oczekiwane, przejrzane komendy i czy jest bezpieczny lokalnie oraz w CI/CD?

## `--ignore-scripts`

W jobie lint widoczny był wzorzec:

```bash
npm install --ignore-scripts
cd frontend
npm install --ignore-scripts
```

Nie wiedziałem początkowo dokładnie, co robi `--ignore-scripts`, więc to sprawdziłem.

Nauczyłem się, że instaluje dependencies, ale blokuje lifecycle scripts takie jak:

```text
preinstall
install
postinstall
prepare
```

To może być dobra kontrola, gdy job potrzebuje zależności, ale nie potrzebuje uruchamiać install scripts.

To nie jest wzorzec, który widzę często w małych codziennych frontendowych `package.json`, ale ma sens w bardziej security-aware CI/CD.

Ważny niuans:

`--ignore-scripts` zmniejsza ryzyko install-time script execution tam, gdzie jest użyte.

Nie usuwa ryzyka wszędzie, jeśli inne joby nadal wykonują zwykłe `npm install`.

W reviewowanym workflow część jobów używała `--ignore-scripts`, a inne test/build/package jobs używały zwykłego `npm install`.

## Lockfile i powtarzalność

Podczas review nie widziałem normalnego commitowanego root `package-lock.json` ani frontend `package-lock.json` w aktywnym użyciu, podczas gdy CI wykonywało powtarzające się kroki `npm install`.

To jest ważne, bo bez commitowanego lockfile dependency resolution jest mniej deterministyczne.

Build dzisiaj i build w przyszłości mogą pobrać inne wersje transitive dependencies bez żadnej zmiany w kodzie źródłowym.

To był jeden z najmocniejszych punktów review A03.

## `npm install` vs `npm ci`

Podczas tego review lepiej zrozumiałem różnicę.

`npm install`:

- jest częste lokalnie,
- instaluje dependencies,
- może aktualizować lockfile, jeśli istnieje,
- może dynamicznie rozwiązywać wersje z zakresów.

`npm ci`:

- jest zaprojektowane do CI,
- wymaga lockfile,
- instaluje dokładnie z lockfile,
- failuje, jeśli `package.json` i lockfile są niespójne,
- jest zwykle lepsze dla powtarzalnych buildów CI.

## SBOM

Projekt miał konfigurację SBOM używającą CycloneDX.

Po raz pierwszy zobaczyłem SBOM generation w praktyce.

SBOM oznacza Software Bill of Materials. To inventory komponentów zawartych w aplikacji albo artefakcie.

SBOM pomaga odpowiedzieć na pytania:

- Jakie dependencies są w artefakcie?
- Czy używamy paczki dotkniętej nową podatnością?
- Które komponenty trzeba sprawdzić podczas incydentu?
- Co faktycznie znalazło się w release package?

Ważne doprecyzowanie:

> SBOM nie naprawia podatności sam z siebie. Daje widoczność. Jest mocniejszy razem z dependency scanning, vulnerability databases, alertingiem i procesem remediation.

## GitHub Actions pinning

Wiele GitHub Actions w workflow było przypiętych do commit SHA zamiast tylko do ruchomych tagów.

To dobra kontrola supply chain, bo poprawia powtarzalność i zmniejsza ryzyko cichego pobrania zmienionego kodu akcji przez mutable tags.

Przykładowy wzorzec:

```yaml
uses: actions/checkout@<commit-sha> # version comment
```

Część akcji wyglądała jednak na użyte przez tagi zamiast commit SHA.

To nie jest automatycznie krytyczny problem, ale jest to punkt review, bo tagi mogą zmienić wskazywany kod.

Silniejszy wzorzec to pinning do przejrzanego SHA i świadome aktualizacje.

## `curl | sh`

Workflow zawierał remote install pattern podobny do:

```bash
curl https://cli-assets.heroku.com/install.sh | sh
```

To klasyczny punkt review CI/CD supply chain.

Ryzyko polega na tym, że pipeline pobiera zdalny skrypt i od razu go wykonuje.

To wymaga szczególnego review, zwłaszcza jeśli job ma dostęp do deployment secrets.

Lepsze kontrole mogą obejmować:

- pinned installer versions,
- checksum verification,
- signature verification,
- instalację przez zaufany package manager,
- ograniczone job permissions,
- ograniczenie sekretów dostępnych dla joba.

## Supply chain review to nie tylko podatne paczki

Przed tym review kojarzyłem dependency security głównie ze znanymi podatnymi paczkami.

To review pokazało mi, że A03 obejmuje też:

- install scripts,
- powtarzalność buildów,
- bezpieczeństwo CI/CD workflow,
- GitHub Actions pinning,
- SBOM,
- release artifacts,
- remote installer scripts,
- sekrety w pipeline contexts.

## Kontrole, których nie widzę codziennie we frontendzie

Nie widzę często SBOM generation ani `--ignore-scripts` w małych codziennych projektach frontendowych.

To review pomogło mi zrozumieć, kiedy mają sens:

- `--ignore-scripts` ma sens, gdy job CI nie potrzebuje lifecycle scripts.
- SBOM ma sens, gdy projekt chce widzieć komponenty w artefakcie.
- SHA-pinning GitHub Actions ma sens, gdy projekt chce mocniejszej powtarzalności CI/CD.

## Największy punkt review

Najmocniejszym problemem nie była jedna konkretna paczka.

Najmocniejszy problem wynikał z połączenia:

- braku aktywnego commitowanego lockfile,
- powtarzającego się `npm install`,
- dependency ranges, które mogą rozwiązać się inaczej w czasie.

To może osłabiać powtarzalność buildów i zwiększać ekspozycję na nieoczekiwane zmiany dependency.

## Dobre kontrole mogą istnieć obok ryzyk

Review nie powinno wypisywać tylko problemów.

W projekcie widoczne były pozytywne kontrole:

- SBOM generation,
- częściowe użycie `--ignore-scripts`,
- wiele GitHub Actions przypiętych do SHA,
- gated Docker publishing,
- uporządkowane test/build workflows.

Jednocześnie nadal istniały punkty review:

- zwykłe `npm install` w kilku jobach,
- missing lockfile/reproducibility concerns,
- `curl | sh`,
- część akcji użyta przez tag.

## Real-world review angle

W realnym projekcie użyłbym tego patternu A03 do sprawdzenia:

- czy istnieje commitowany lockfile,
- czy CI używa `npm ci`, gdzie to możliwe,
- czy lifecycle scripts są potrzebne w CI,
- czy jakiś job może użyć `--ignore-scripts`,
- czy GitHub Actions są przypięte do SHA czy tylko tagów,
- czy third-party actions są zaufane i przejrzane,
- czy sekrety są dostępne dla jobów uruchamiających dependency scripts,
- czy dependency scanning jest włączony,
- czy secret scanning jest włączony,
- czy SBOM jest generowany dla release artifacts,
- czy Docker base images są przypięte lub reviewowane,
- czy release artifacts są powiązane z testowanym kodem,
- czy `curl | sh` jest unikane albo weryfikowane.

## Podsumowanie

Najważniejsze lekcje A03 z tej praktyki:

1. `postinstall` i lifecycle scripts są ważnym punktem review supply chain.
2. `--ignore-scripts` może ograniczać install-time code execution, gdy jest użyte świadomie.
3. Brak lockfiles może osłabiać powtarzalność buildów.
4. `npm ci` jest zwykle mocniejsze niż `npm install` w CI, jeśli istnieje lockfile.
5. SBOM daje widoczność dependencies, ale sam nie naprawia podatności.
6. `curl | sh` wymaga uważnego review w CI/CD.
7. SHA-pinned GitHub Actions poprawiają powtarzalność i zmniejszają ryzyko third-party actions.
