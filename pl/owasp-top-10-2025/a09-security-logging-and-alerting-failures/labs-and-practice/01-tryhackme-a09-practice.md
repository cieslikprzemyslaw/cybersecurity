# TryHackMe — A09 Security Logging and Alerting Failures

## Źródło

- TryHackMe: OWASP Top 10 2025 — IAAA Failures
- Zakres: wyłącznie sekcja A09

## Co zostało ukończone

Przejrzałem teorię i praktykę A09 wraz z oficjalnym OWASP A09 oraz dwoma cheat sheetami OWASP dotyczącymi loggingu.

## Najważniejsze obserwacje

- brak eventu uniemożliwia późniejszą detekcję,
- sam event nie dowodzi, że atak został wykryty,
- ważne są zarówno success, jak i failure security controls,
- alert potrzebuje progu, kontekstu, ownera i response process,
- logi muszą być chronione przed odczytem, zmianą, usunięciem i floodingiem,
- DAST i autoryzowane testy powinny generować oczekiwane eventy i alerty,
- logi mogą ujawniać PII, PHI, tokeny i inne sekrety,
- niebezpiecznie zakodowany input może prowadzić do log injection,
- brak lub nieaktualny playbook tworzy response gap.

## Evidence

Evidence z nauki obejmowało ukończenie sekcji A09, review oficjalnych źródeł oraz kolejne ćwiczenia dotyczące authentication logging, log injection i sensitive data.

## Ograniczenia

Ćwiczenie nie obejmowało konfiguracji produkcyjnego SIEM ani pracy analityka SOC. Celem było application-level review z perspektywy Frontend Developera przechodzącego do AppSec.
