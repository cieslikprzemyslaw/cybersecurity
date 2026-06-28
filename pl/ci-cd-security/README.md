# Bezpieczeństwo CI/CD

Ten dział zawiera praktyczne notatki o bezpieczeństwie CI/CD z perspektywy frontendowej i AppSec.

Nie chodzi tylko o to, żeby GitHub Actions były zielone. Chodzi o zrozumienie, co potok CI/CD może zrobić, do czego ma dostęp, co może pójść źle i dlaczego CI/CD staje się częścią granicy bezpieczeństwa aplikacji.

## Moduły

| Moduł | Zakres |
|---|---|
| [Utwardzanie GitHub Actions](github-actions-hardening/00-index.md) | Utwardzanie GitHub Actions, sprawdzenie działania Dockera, polityka audytu zależności, CodeQL SAST, ochrona repozytorium, cleanup triggerów i higiena sekretów. |

## Kontekst praktyczny

Notatki bazują na realnej pracy w projekcie AppSec Report Builder.

Projekt miał już działający workflow CI. Sprint rozbudował go do warstwowego potoku bezpieczeństwa CI/CD:

```text
Validate
Docker Runtime
Dependency Security
CodeQL / SAST
Repository Protection
Secrets and Repository Hygiene
```

## Zakres nauki

- Traktowanie GitHub Actions jako zaufanego środowiska wykonawczego.
- Ograniczanie uprawnień workflowa do minimum potrzebnego dla joba.
- Walidowanie nie tylko kodu, ale też zachowania kontenera w runtime.
- Zamiana wyniku audytu zależności w czytelną politykę review.
- Używanie SAST jako wejścia do triage, a nie zamiennika review inżynierskiego.
- Utrzymywanie widocznej ochrony repozytorium, triggerów i higieny sekretów.

## Główna lekcja

Pipeline CI/CD nie jest tylko automatyzacją. To zaufany system, który instaluje zależności, uruchamia skrypty, buduje aplikację i pomaga zdecydować, czy zmiana jest wystarczająco bezpieczna do merge.

Dlatego CI/CD jest częścią AppSec.
