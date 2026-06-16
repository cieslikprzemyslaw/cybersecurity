# Monitoring, Alerting i Response

## Log nie jest detekcją

Pojedynczy event może zostać zapisany poprawnie i nadal pozostać niezauważony.

```text
recorded event != detected attack
```

Skuteczna detekcja zwykle wymaga:

```text
events + correlation + rule + threshold + time window + context
```

Skuteczne alertowanie dodatkowo wymaga:

```text
severity + recipient + ownership + playbook + timely review
```

## Projekt reguły detekcji

Przydatna reguła powinna definiować:

1. attack albo abuse pattern,
2. źródło eventu,
3. correlation key,
4. threshold,
5. time window,
6. wyjątki lub klasyfikacje,
7. severity i confidence,
8. kontekst alertu,
9. właściciela i oczekiwaną reakcję.

### Przykład — wielokrotne błędne logowanie

```text
Pattern: authn_login_fail dla tego samego konta
Threshold: 5 eventów
Window: 5 minut
Initial severity: WARN
Recipient: security team albo on-call owner
Response: przejrzeć źródła prób, rate limiting, ryzyko konta i powiązane sesje
```

### Przykład — sukces po wcześniejszych błędach

```text
Pattern: authn_login_success po co najmniej 5 authn_login_fail
Correlation: to samo konto
Window: 10 minut
Severity: HIGH lub wyższe dla privileged account
Recipient: security team albo on-call owner
Response: przejrzeć MFA, sesję, źródło i aktywność po logowaniu; unieważnić sesje przy wysokiej pewności
```

Progi są przykładowe. Muszą być dostrojone do użytkowników, ruchu, ryzyka i sposobu działania lockout/rate limiting.

## Korelacja

Przydatne correlation keys:

- account ID,
- target resource,
- request lub interaction ID,
- source IP jako kontekst,
- cechy urządzenia albo user agenta,
- application i environment,
- time window.

Adres IP nie jest wiarygodną tożsamością użytkownika. NAT, proxy, sieci mobilne, VPN i współdzielone sieci mogą łączyć wielu użytkowników pod jednym adresem albo jednego użytkownika rozdzielać pomiędzy różne adresy.

## Actionable alerts

Alert powinien zawierać wystarczający kontekst do podjęcia decyzji:

- nazwę eventu i reguły,
- konto lub zasób,
- istotną sekwencję eventów,
- zakres czasu,
- source context,
- confidence i severity,
- aktualny stan konta lub sesji,
- guidance do response,
- link do dowodów,
- właściciela albo ścieżkę eskalacji.

Alert bez kontekstu opóźnia dochodzenie. Alert bez właściciela staje się nieobsłużonym zadaniem.

## False positives i false negatives

### True positive

Reguła poprawnie identyfikuje podejrzane albo złośliwe zachowanie.

### False positive

Alert uruchamia się dla legalnego albo nieistotnego bezpieczeństwowo zachowania.

### False negative

Podejrzane albo złośliwe zachowanie występuje, ale reguła nie uruchamia alertu.

### Accepted risk

Rzeczywiste ryzyko jest udokumentowane i zaakceptowane przez uprawnionego risk ownera. To nie jest false positive.

## Alert fatigue

Alertowanie każdego pojedynczego błędu tworzy szum. Nadmiar alertów może spowodować, że istotne zdarzenia zostaną przejrzane zbyt późno albo zignorowane.

Kontrole:

- korelacja zamiast jednego alertu na event,
- risk-based thresholds,
- kontekst konta i zasobu,
- deduplication i suppression windows,
- tuning oparty na dowodach,
- regularny review reguł i playbooków.

## Response flow

High-confidence alert może prowadzić do:

1. triage i walidacji,
2. zabezpieczenia dowodów,
3. unieważnienia sesji albo innego containment,
4. ochrony konta,
5. review aktywności po kompromitacji,
6. naprawy pierwotnej słabości,
7. recovery i komunikacji,
8. poprawy reguł i playbooka.

Logging i alerting nie zastępują prevention. Pozwalają wykryć próbę lub udane nadużycie i podjąć działanie.

## Honeytokens

Honeytoken jest przynętą — wartością lub tożsamością, która nie powinna być używana w normalnym procesie biznesowym. Dostęp może wygenerować event o wysokiej pewności i niewielkiej liczbie legalnych wyjaśnień.

Honeytokens nadal wymagają:

- chronionego umieszczenia,
- wiarygodnego źródła eventu,
- przetestowanego alertu,
- właściciela i playbooka,
- bezpiecznego projektu bez szkody dla działania aplikacji.

## Weryfikacja

Detekcja nie jest kompletna, dopóki nie zostanie przetestowana pełna ścieżka:

```text
test action
    -> expected event emitted
    -> event collected
    -> rule matched
    -> alert delivered
    -> alert contains useful context
    -> owner can follow the playbook
```

DAST, penetration testing i kontrolowane testy abuse case powinny generować oczekiwane eventy i alerty. Brak reakcji jest dowodem detection gap.
