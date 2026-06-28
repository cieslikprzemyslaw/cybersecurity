# Utwardzanie GitHub Actions

## Cel

Ten moduł opisuje praktyczny sprint bezpieczeństwa CI/CD wykonany w projekcie AppSec Report Builder.

Celem było przejście od zwykłego workflow walidacyjnego do warstwowego potoku, który lepiej wspiera bezpieczeństwo.

Końcowy pipeline obejmował:

```text
Validate
Docker Runtime
Dependency Security
CodeQL / SAST
Repository Protection
Secrets and Repository Hygiene
```

## Podsumowanie dowodów

Pełna lista PR-ów i końcowy status CI są w pliku [04 Dowody](04-dowody.md). Końcowy screenshot znajduje się w [ci-cd-security-github-actions-success.png](../../../assets/ci-cd-security-github-actions-success.png).

Dowody pokazują poprawne uruchomienie `ci.yml` po merge, w tym walidację kodu, sprawdzenie runtime Dockera i kontrolę bezpieczeństwa zależności.

## Pliki w module

| Plik | Cel |
|---|---|
| [01 Model myślenia o pipeline](01-model-mentalny-pipeline.md) | Jak patrzeć na CI/CD jako część powierzchni ataku. |
| [02 Lab AppSec Report Builder](02-lab-appsec-report-builder.md) | Praktyczny opis wdrożonych kontroli. |
| [03 Checklista](03-checklista.md) | Checklista do kolejnych review CI/CD. |
| [04 Dowody](04-dowody.md) | Numery PR-ów, końcowy status i notatki dowodowe. |

## Główny wniosek

Bezpieczny pipeline CI/CD to nie jedno narzędzie. To łańcuch małych kontroli, które utrudniają ciche przepchnięcie ryzykownej zmiany.

## Źródła

- OWASP CI/CD Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html
- GitHub Actions security hardening: https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions
- Dokumentacja GitHub CodeQL: https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql
- GitHub secret scanning: https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning
