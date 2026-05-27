# Przykładowe znalezisko: niepowtarzalne instalacje npm zwiększają ryzyko supply chain

## Podsumowanie

Review supply chain wykazało, że projekt wydaje się polegać na dynamicznej instalacji zależności npm w CI/CD bez jasno egzekwowanej strategii commitowanego lockfile.

Kilka jobów używa `npm install`, a review nie potwierdziło aktywnie używanego lockfile dla root ani frontend. To może sprawić, że dependency resolution będzie mniej deterministyczne i transitive dependencies zmienią się między buildami bez zmiany kodu źródłowego.

To znalezisko jest przykładem edukacyjnym na podstawie publicznego practice review, nie disclosure wobec reviewowanego projektu.

## Dotknięty obszar

Dotknięte obszary:

```text
package.json
frontend/package.json
GitHub Actions workflows
CI dependency installation steps
build/test/package jobs
```

## Mapowanie OWASP

```text
OWASP Top 10 2025: A03 - Software Supply Chain Failures
```

Powiązane typy słabości:

```text
Non-reproducible dependency installation
Uncontrolled dependency resolution
CI/CD supply chain risk
Dependency management weakness
```

## Ryzyko / wpływ

Bez commitowanego lockfile i powtarzalnego install strategy dependency tree może zmieniać się w czasie mimo braku zmian w aplikacji.

Możliwy wpływ:

- przyszłe buildy instalują inne transitive dependency versions,
- nowo podatna albo skompromitowana paczka trafia do builda,
- debugowanie i incident response są trudniejsze,
- artefakt testowany może być trudny do odtworzenia,
- dependency changes są mniej widoczne w PR,
- auditability procesu build/release jest słabsza.

To nie oznacza automatycznie kompromitacji aplikacji. Problemem jest słabsza kontrola nad nieoczekiwanymi zmianami zależności.

## Root cause

Główną przyczyną jest brak jasno egzekwowanej strategii powtarzalnej instalacji zależności.

Czynniki z review:

- brak aktywnie commitowanego root lockfile,
- brak aktywnie commitowanego frontend lockfile,
- wiele użyć `npm install` w CI,
- version ranges mogące rozwiązywać się inaczej w czasie,
- lifecycle scripts uruchamiane w jobach używających normalnego `npm install`.

## Evidence

Workflow CI zawierał kroki instalacji zależności przez `npm install`:

```yaml
- name: "Install application"
  run: npm install
```

W tym samym pipeline istniał też bezpieczniejszy wzorzec w jobie lint:

```bash
npm install --ignore-scripts
cd frontend
npm install --ignore-scripts
```

To ogranicza ryzyko install-time script execution w wybranych jobach, ale nie zastępuje lockfile-based reproducibility.

## Pozytywne kontrole

Review wykazało też dobre elementy:

- część jobów używa `npm install --ignore-scripts`,
- wiele GitHub Actions jest pinned to commit SHA,
- obecne jest SBOM generation przez CycloneDX,
- Docker publishing jest ograniczony do trusted context,
- workflow test/build/package są uporządkowane.

## Oczekiwane bezpieczne zachowanie

Bezpieczniejszy proces powinien:

- commitować lockfiles,
- używać `npm ci` w CI tam, gdzie można,
- reviewować lockfile diffs,
- wyłączać lifecycle scripts tam, gdzie nie są potrzebne,
- mieć dependency scanning,
- generować SBOM w release process,
- pozwalać prześledzić artefakt do testowanego kodu i wersji zależności.

## Remediation

1. Dodać i commitować właściwe lockfiles, np. `package-lock.json` i `frontend/package-lock.json`.
2. Używać `npm ci` w jobach CI/CD, gdy lockfile jest dostępny.
3. Udokumentować joby, które muszą używać `npm install`.
4. Kontynuować `--ignore-scripts` tam, gdzie lifecycle scripts nie są potrzebne.
5. Sprawdzić joby, które używają normalnego `npm install` i mają dostęp do sekretów.
6. Włączyć dependency scanning albo potwierdzić aktywne alerts.
7. Utrzymać SBOM generation w build/release.
8. Reviewować lockfile changes w PR.
9. Przypisać ownership dla dependency updates.
10. Rozważyć artifact provenance/attestation dla release.

## Pomysły na testy regresji

```text
Lockfiles are present for npm projects.
CI uses npm ci where lockfiles exist.
CI fails if package.json and package-lock.json are out of sync.
Dependency install jobs do not unexpectedly run lifecycle scripts where not needed.
SBOM generation still runs during package/release.
Dependency scanning runs before release.
Deployment jobs do not install unreviewed dependency versions dynamically.
```

## Developer takeaway

Supply chain security to nie tylko pytanie "czy paczka ma CVE?". W projektach Node.js i frontendowych trzeba też pytać:

- czy dependency versions są powtarzalne,
- czy install scripts uruchamiają się automatycznie,
- czy CI używa lockfile,
- czy build/release artifacts są traceable,
- czy zmiany zależności są widoczne w review.

Lockfile sam nie czyni projektu bezpiecznym, ale jest ważną kontrolą dla reproducibility i dependency visibility.
