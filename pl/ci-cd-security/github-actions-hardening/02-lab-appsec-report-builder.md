# Lab: bezpieczeństwo CI/CD w AppSec Report Builder

## Cel

Celem było poprawienie pipeline w projekcie AppSec Report Builder małymi, łatwymi do sprawdzenia krokami.

Projekt miał już działający workflow CI. Sprint dodał wokół niego warstwy bezpieczeństwa i niezawodności.

## Punkt startowy

Istniejący workflow robił już sensowną walidację:

```text
npm ci
format check
lint
typecheck
testy
build
```

To była dobra baza, ale nadal brakowało odpowiedzi na pytania bezpieczeństwa:

```text
Czy workflow ma minimalne uprawnienia?
Czy obraz Dockera faktycznie startuje?
Czy podatności high/critical w zależnościach blokują PR?
Czy mamy SAST dla JavaScript/TypeScript?
Czy można zmienić master bez review?
Czy sekrety są chronione przed przypadkowym commitem?
Czy podwójne uruchomienia CI nie robią szumu?
```

---

## Krok 1: utwardzenie walidacji CI

PR:

```text
#365 chore: harden CI validation workflow
```

Zmiany obejmowały:

```text
permissions: contents: read
timeout-minutes: 15
concurrency z cancel-in-progress
jawna walidacja Prisma
jawne generowanie Prisma
```

### Dlaczego to miało znaczenie

Job nie potrzebował uprawnień zapisu, więc `contents: read` było wystarczające.

Timeout i concurrency poprawiły stabilność workflowa i zmniejszyły szum.

Kroki Prisma sprawiły, że walidacja schematu bazy była jawna, a nie ukryta dopiero w późniejszym błędzie builda albo testów.

### Wynik

```text
CI/CD Security #1 — CI validation hardening: PASS
```

---

## Krok 2: sprawdzenie działania Dockera

PR:

```text
#366 chore: validate Docker runtime in CI
```

Dodatkowy kontekst obrazu Dockera:

```text
#364 Chore/docker api image
```

Nowy job budował obraz Docker dla API i uruchamiał prawdziwy kontener w CI.

Następnie sprawdzał:

```text
/api/health
```

Job wypisywał też logi Dockera przy błędzie i sprzątał kontener po zakończeniu.

### Realny problem znaleziony w CI

Pierwsze sprawdzenie działania kontenera padło, bo Prisma oczekiwała `DATABASE_URL` przed stworzeniem Prisma Client.

Poprawka polegała na uruchomieniu kontenera z:

```text
DATABASE_URL=file:/data/appsec-report-builder.db
```

### Dlaczego to miało znaczenie

To była realna lekcja CI/CD security i niezawodności.

Obraz kontenera może się zbudować poprawnie, ale nadal nie uruchomić się przez brak konfiguracji.

### Wynik

```text
CI/CD Security #2 — Docker runtime validation: PASS
```

---

## Krok 3: polityka audytu zależności

PR:

```text
#367 chore: add dependency audit policy
```

Zamiast zwykłego `npm audit --audit-level=high` projekt dostał własną politykę opartą o `npm audit --json`.

Polityka blokuje podatności high i critical, chyba że są opisane jako tymczasowo zaakceptowane ryzyko.

Poprawna tymczasowa akceptacja wymaga:

```text
pakietu
poziomu ryzyka
powodu
daty wygaśnięcia
follow-up issue
opcjonalnego advisory ID
```

### Realny problem znaleziony i naprawiony

Audit znalazł krytyczny łańcuch:

```text
concurrently -> shell-quote
```

`concurrently` było direct dependency, czyli zależnością zadeklarowaną bezpośrednio w projekcie. `shell-quote` było transitive dependency, czyli zależnością pobraną pośrednio.

Decyzja:

```text
napraw teraz
```

Powód:

```text
direct dependency miało dostępną poprawkę
```

### Dlaczego to miało znaczenie

Powstał praktyczny proces triage:

```text
pojawia się nowe high/critical
CI blokuje PR
engineer sprawdza wynik
naprawia albo opisuje tymczasową akceptację ryzyka
zaakceptowane ryzyko musi mieć datę wygaśnięcia
```

### Wynik

```text
CI/CD Security #3 — Dependency audit policy: PASS
```

---

## Krok 4: CodeQL SAST

PR:

```text
#368 chore: add CodeQL SAST workflow
```

CodeQL został dodany dla JavaScript/TypeScript.

Pierwszy cel to widoczność i triage, a nie paniczne blokowanie wszystkiego od pierwszego dnia.

### Dlaczego to miało znaczenie

SAST może znaleźć ryzykowne wzorce w kodzie, ale wyniki wymagają kontekstu.

Na przykład SAST może pomóc przy wzorcach podobnych do injection albo przy niebezpiecznym użyciu API. Słabiej radzi sobie z logiką biznesową, np. IDOR albo broken authorization.

### Wynik

```text
CI/CD Security #4 — CodeQL SAST visibility: PASS
```

---

## Krok 5: cleanup triggerów CI

PR:

```text
#369 chore: avoid duplicate CI runs for PR branches
```

Trigger CI został zmieniony tak, żeby:

```text
pull request do master uruchamiał CI przez pull_request
push do master uruchamiał CI po merge
zwykły push do brancha z PR nie tworzył dodatkowego duplikatu
```

### Dlaczego to miało znaczenie

To jest porządek w pipeline.

Zmniejsza zużycie runner minutes, ułatwia czytanie wyników PR i usuwa podwójne statusy, które robiły szum.

### Wynik

```text
Trigger cleanup: PASS
```

---

## Krok 6: ochrona repozytorium

Repozytorium już blokowało direct push do `master`, nawet dla ownera.

To oznacza, że normalna ścieżka wygląda tak:

```text
branch
pull request
checki
merge
```

### Dlaczego to miało znaczenie

Ochrona gałęzi zmienia CI z pomocnego narzędzia w wymuszoną bramkę review.

Zapobiega przypadkowemu albo celowemu ominięciu walidacji.

### Wynik

```text
Repository protection: PASS
```

---

## Krok 7: higiena sekretów i repozytorium

Repozytorium zostało sprawdzone pod kątem podstawowej higieny sekretów.

Potwierdzone:

```text
.env jest ignorowany
lokalne pliki bazy danych są ignorowane
logi są ignorowane
foldery uploadów są ignorowane
lokalne sekrety Dockera są ignorowane
workflow nie potrzebuje sekretów
lokalny scan pod kątem sekretów przeszedł
```

### Dlaczego to miało znaczenie

Sekrety nie powinny trafiać do kontroli wersji.

Nawet projekt local-first nie powinien commitować lokalnych baz, logów, uploadów, plików `.env` ani sekretów Dockera.

### Wynik

```text
CI/CD Security #5 — Secrets and repository hygiene: PASS
```

---

## Końcowy wynik

Końcowe dowody CI są opisane w [04 Dowody](04-dowody.md). Najważniejszy wynik: scalony workflow przeszedł główne warstwy bezpieczeństwa CI/CD:

```text
Validate
Docker Runtime
Dependency Security
```

Szerszy sprint dodał też:

```text
CodeQL / SAST
Repository Protection
Secrets Hygiene
Trigger cleanup
```

## Podsumowanie techniczne

```text
Poprawiłem pipeline CI/CD w AppSec Report Builder przez dodanie minimalnych uprawnień workflowa, sprawdzenia działania kontenera Docker, polityki audytu zależności, CodeQL SAST, ochrony gałęzi, cleanup triggerów i podstawowej higieny sekretów w repozytorium.
```
