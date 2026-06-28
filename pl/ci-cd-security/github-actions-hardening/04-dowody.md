# Dowody

## Scalone PR-y

Sprint CI/CD Security został wdrożony przez małe, scalone pull requesty.

| PR | Tytuł | Wartość dowodowa |
|---|---|---|
| #365 | chore: harden CI validation workflow | Dodano utwardzenie CI: minimalne uprawnienia, timeout, concurrency i jawne kroki walidacyjne. |
| #366 | chore: validate Docker runtime in CI | Dodano build obrazu, start kontenera, health check, logi przy błędzie i cleanup. |
| #367 | chore: add dependency audit policy | Dodano politykę opartą o npm audit JSON i blokadę high/critical bez udokumentowanej akceptacji ryzyka. |
| #368 | chore: add CodeQL SAST workflow | Dodano CodeQL jako SAST dla JavaScript/TypeScript. |
| #369 | chore: avoid duplicate CI runs for PR branches | Uporządkowano triggery CI, żeby ograniczyć podwójne uruchomienia dla branchy z PR. |

Dodatkowy kontekst:

| PR | Tytuł | Wartość dowodowa |
|---|---|---|
| #364 | Chore/docker api image | Dostarczył kontekst obrazu Dockera, który później został użyty w walidacji runtime. |

## Końcowy screenshot CI

Plik screenshota:

[ci-cd-security-github-actions-success.png](../../../assets/ci-cd-security-github-actions-success.png)

Screenshot pokazuje:

```text
Workflow: ci.yml
Trigger: push do master
Status: Success
Czas całkowity: 3m 59s
Validate: 2m 57s
Docker Runtime: 54s
Dependency Security: 25s
Vitest: 138 plików testowych, 545 testów, wszystko przeszło
```

## Co to potwierdza

Screenshot potwierdza, że po scaleniu zmian końcowy pipeline poprawnie uruchomił główne warstwy walidacji:

```text
Validate
Docker Runtime
Dependency Security
```

Lista PR-ów potwierdza, że CodeQL i cleanup triggerów zostały dodane jako osobne, sprawdzone zmiany.

## Dowód ochrony repozytorium

Ochrona repozytorium została potwierdzona przez test, że direct push do `master` jest zablokowany nawet dla ownera.

To oznacza, że zmiany muszą iść ścieżką:

```text
branch -> pull request -> checki -> merge
```

## Dowód higieny sekretów

Lokalne review potwierdziło:

```text
.env jest ignorowany
lokalne pliki bazy danych są ignorowane
logi są ignorowane
uploady są ignorowane
lokalne sekrety Dockera są ignorowane
workflow nie potrzebuje sekretów
lokalny scan pod kątem sekretów przeszedł
```

## Końcowe zdanie dowodowe

```text
Projekt AppSec Report Builder ma teraz warstwową bazę bezpieczeństwa CI/CD: walidację kodu, sprawdzenie działania Dockera, politykę podatności zależności, CodeQL SAST, chroniony przepływ przez PR, czystsze triggery i podstawową higienę sekretów w repozytorium.
```
