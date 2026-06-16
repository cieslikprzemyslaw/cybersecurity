# Learning Notes

## Początkowe rozumienie

Rozumiałem, że logi pomagają wykrywać podejrzaną aktywność, ale mój pierwszy model był niepełny.

Początkowo myślałem o loggingu głównie jako o osobnym produkcie monitorującym, a backend miał po prostu utworzyć log i wysłać go do bazy danych. Poprawiony model:

```text
application or security-control component
    -> structured event
    -> collector and protected storage
    -> detection and correlation
    -> alert
    -> response
```

Baza danych może być jednym z mechanizmów przechowywania, ale nie definiuje loggingu.

## Logging, monitoring, alerting i response

Moje pierwsze definicje były zbyt krótkie:

- logging — tworzenie wpisu,
- monitoring — obserwowanie zdarzeń,
- alerting — powiadomienie o potencjalnie niebezpiecznym zdarzeniu,
- incident response — działanie po wykryciu.

Korekta: monitoring obejmuje również collection, analysis i correlation. Alerting wymaga zdefiniowanego warunku, kontekstu, ownera i oczekiwanej akcji. Incident response obejmuje triage, containment, remediation, recovery, evidence preservation i learning.

## Ćwiczenie z authentication logging

Prawidłowo wskazałem Auth Service jako autorytatywne źródło decyzji login i MFA.

Moja pierwsza lista eventów była zbyt ogólna:

```text
authn_{login/mfa}_{failed/success/recovery/reset}
```

Lepsze podejście używa konkretnych eventów:

```text
authn_login_fail
authn_login_success
authn_mfa_fail
authn_mfa_success
session_created
```

Recovery albo reset powinny być emitowane wyłącznie wtedy, gdy rzeczywiście występuje taki workflow.

Dowiedziałem się też, że review authentication wymaga czegoś więcej niż login events. Session creation, renewal, invalidation, password-reset token validation i użycie expired albo revoked session mogą być istotne dla bezpieczeństwa.

## Detection gap

Prawidłowo zauważyłem, że zapis nieudanych logowań nie dowodzi detekcji brute force.

Najważniejsza luka:

> Eventy mogą istnieć, ale żadna reguła ich nie koreluje, alert nie zostaje dostarczony albo nikt nie jest odpowiedzialny za reakcję.

Analizowane wzorce:

- wiele prób przeciwko jednemu kontu,
- jedno źródło próbujące wielu kont,
- distributed attempts przeciwko jednemu kontu,
- successful login po wielu failures,
- powtarzające się MFA failures.

Skorygowałem też niebezpieczny przykład z hasłem w URL. Credentials nie powinny trafiać do query string, ponieważ URL może zostać zapisany w historii, proxy, access logs, analytics i referrer data.

## Ćwiczenie log injection

Rozpoznałem główny przepływ:

```text
user-controlled username
    -> interpolated log line
    -> możliwa manipulacja danymi logu
```

Użyłem określenia, że atakujący może „nadpisać” log. W analizowanym przykładzie bardziej precyzyjny skutek to wstrzyknięcie wpisu wyglądającego jak dodatkowy event, uszkodzenie struktury albo wprowadzenie parsera w błąd. Nadpisanie wcześniejszych rekordów zwykle wymaga dodatkowego dostępu albo innej słabości.

Nauczyłem się też rozdzielać:

- mechanizm — manipulacja wpisem,
- skutek — błędna interpretacja przez parser, SIEM, regułę alertu albo analityka.

Skutkiem mogą być false negatives i false positives.

## Ćwiczenie sensitive logging

Prawidłowo zaklasyfikowałem całe request body i session cookie jako dane, których nie należy logować.

Początkowo uznałem:

```text
actorUserId -> do not log
targetUserId -> do not log
```

To było nieprawidłowe dla audit eventu zmiany roli. Wewnętrzne identyfikatory aktora i celu są kluczowym dowodem, kto zmienił czyje uprawnienia.

Access token zaklasyfikowałem jako „masked or transformed”, ponieważ myślałem, że hash może pomóc sprawdzić CSRF albo inną kontrolę. Korekta:

- nie logować access tokenu,
- nie logować CSRF tokenu,
- logować rezultat kontroli, np. `csrf_validation_fail`,
- korelować przy użyciu request, interaction lub server-generated reference.

Hashing nie jest automatyczną ochroną sekretów.

## Minimalny audit event zmiany roli

```ts
const auditEvent = {
  timestamp: new Date().toISOString(),
  eventName: "privilege_permissions_changed",
  actorUserId: authenticatedUser.id,
  targetUserId: request.body.userId,
  previousRole,
  newRole: request.body.role,
  result: "success",
  requestId: request.id,
};
```

Główny dowód:

```text
who + target + previous state + new state + time + result
```

Source IP i user agent mogą wspierać dochodzenie, ale nie dowodzą tożsamości.

## Korekta severity

Początkowo opisałem severity głównie jako wpływ: low, medium, high albo critical.

Lepsze rozumienie: severity wyraża znaczenie i priorytet reakcji w danym kontekście. Zależy między innymi od:

- impact,
- asset sensitivity,
- event result,
- confidence,
- frequency,
- account privilege,
- wymaganej szybkości reakcji.

`eventName`, `result` i `severity` pozostają osobnymi polami.

## Końcowy wniosek

> Security logging jest skuteczny tylko wtedy, gdy aplikacja emituje właściwy i bezpieczny event, rekord pozostaje wiarygodny i dostępny, podejrzane wzorce są wykrywane, actionable alert trafia do właściciela, a reakcję można wykonać i zweryfikować.
