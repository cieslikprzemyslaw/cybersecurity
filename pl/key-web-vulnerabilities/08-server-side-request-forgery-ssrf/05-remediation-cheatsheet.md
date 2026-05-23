# SSRF - ściąga z remediacji

## Cel

Używaj tej notatki, gdy myślisz jak zapobiec SSRF jako developer albo jak napisać rekomendację remediacji w znalezisku.

Naprawy SSRF powinny skupiać się na kontroli tego, dokąd backend może wysyłać requesty.

## Główna zasada

Nie pozwalaj użytkownikom kontrolować dowolnych miejsc docelowych requestów backendowych.

Jeżeli aplikacja musi pobierać zasoby zewnętrzne albo wewnętrzne, dozwolone cele powinny być ściśle kontrolowane i walidowane.

## Bezpieczniejsze wzorce projektowe

### Preferuj ID zamiast URL-i

Zamiast przyjmować:

```json
{
  "stockApi": "http://internal-stock-api/product/stock/check"
}
```

lepiej przyjmować bezpieczny identyfikator:

```json
{
  "storeId": "glasgow-01"
}
```

Następnie mapuj ten identyfikator po stronie serwera na znany bezpieczny endpoint backendowy.

### Używaj mapowań po stronie serwera

Przykład:

```text
storeId=1 -> http://stock-service-1.internal
storeId=2 -> http://stock-service-2.internal
```

Użytkownik kontroluje tylko ID, a nie URL.

### Używaj ścisłych allowlist

Jeżeli URL-e muszą być przyjmowane, pozwalaj tylko na konkretne zaufane cele.

Waliduj:

- scheme,
- hostname,
- rozwiązany IP,
- port,
- ścieżkę, jeśli trzeba,
- redirecty,
- finalny cel.

Unikaj luźnych kontroli typu:

```text
url contains "trusted-site.com"
url starts with "https://trusted-site.com"
```

Takie kontrole można obejść przez sztuczki z parserami URL.

## Blokuj niebezpieczne cele

Backend nie powinien móc odpytywać:

```text
localhost
127.0.0.0/8
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
169.254.0.0/16
169.254.169.254
internal admin hosts
```

Uwzględnij też odpowiedniki IPv6:

```text
::1
fc00::/7
fe80::/10
```

## Waliduj finalny cel

Walidacja powinna odbywać się po:

- parsowaniu URL,
- dekodowaniu URL,
- DNS resolution,
- redirectach,
- canonicalisation / normalisation.

Nie waliduj tylko surowego stringa wejściowego.

Ważne pytanie:

```text
Where will the HTTP client actually connect?
```

To finalny cel musi być bezpieczny.

## Obsługuj redirecty ostrożnie

Jeżeli redirecty są dozwolone:

- waliduj każdy redirect hop,
- ponownie rozwiązuj i sprawdzaj finalny host/IP,
- blokuj redirecty do wewnętrznych zakresów,
- ogranicz liczbę redirectów,
- unikaj downgrade'u protokołu albo nieoczekiwanej zmiany protokołu.

Jeżeli redirecty nie są potrzebne, wyłącz je.

## Ogranicz protokoły i porty

Pozwalaj tylko na wymagane scheme:

```text
https
http tylko jeśli naprawdę potrzebny
```

Blokuj nieoczekiwane scheme:

```text
file
gopher
ftp
dict
ldap
```

Ogranicz porty do oczekiwanych wartości:

```text
443
80 jeśli potrzebny
konkretne znane porty usług wewnętrznych tylko jeśli wymagane
```

## Zabezpieczenia sieciowe

Walidacja aplikacyjna nie wystarczy.

Używaj kontroli infrastrukturalnych:

- reguł egress firewall,
- segmentacji sieci,
- service mesh policies,
- ochrony endpointów metadata,
- cloud IAM least privilege,
- blokowania dostępu do wewnętrznych usług admina z workloadów aplikacji, jeśli nie jest wymagany.

## Chroń cloud metadata

Jeżeli aplikacja działa w cloud:

- blokuj dostęp do metadata endpoint, jeśli nie jest potrzebny,
- używaj zabezpieczeń metadata oferowanych przez providera,
- wymagaj tokenów sesyjnych tam, gdzie są wspierane,
- stosuj least privilege dla ról instancji,
- monitoruj dostęp do metadata.

Wysokiego ryzyka endpoint:

```text
169.254.169.254
```

## Nie ufaj samym requestom wewnętrznym

Wewnętrzne panele admina nadal muszą egzekwować authentication i authorization.

Nie polegaj tylko na tym, że:

```text
request comes from localhost
request comes from internal IP
request comes from trusted network
```

SSRF może sprawić, że złośliwy request będzie wyglądał, jakby pochodził z zaufanego źródła wewnętrznego.

## Logowanie i monitoring

Loguj podejrzane outbound requesty backendu:

- requesty do `localhost`,
- requesty do prywatnych zakresów IP,
- requesty do endpointów metadata,
- nietypowe porty,
- wiele nieudanych requestów wewnętrznych,
- redirect chains do nieoczekiwanych hostów.

Monitoring nie zastępuje prewencji, ale pomaga wykrywać próby exploitacji.

## Pomysły na testy regresji

Po naprawie SSRF dodaj testy dla:

```text
http://localhost/admin
http://127.0.0.1/admin
http://127.1/admin
http://10.0.0.1/
http://172.16.0.1/
http://192.168.0.1/
http://169.254.169.254/
encoded internal hosts
double-encoded paths
redirect to internal host
allowed-host@blocked-host
blocked-host#allowed-host
allowed-host.blocked-host
unexpected ports
```

Oczekiwany wynik:

```text
Backend nie może wysyłać requestów do zablokowanych celów wewnętrznych.
```

## Podsumowanie remediacji dla developerów

Dobra notatka remediacyjna powinna mówić:

1. Nie przyjmuj dowolnych URL-i kontrolowanych przez użytkownika dla requestów backendowych.
2. Zastąp URL-e kontrolowane przez użytkownika mapowaniami po stronie serwera, jeśli to możliwe.
3. Używaj ścisłej allowlisty oczekiwanych celów.
4. Waliduj finalny rozwiązany cel, nie tylko surowy input.
5. Blokuj zakresy loopback, private, link-local i metadata.
6. Sprawdzaj każdy redirect hop.
7. Ogranicz scheme i porty.
8. Dodaj egress controls na poziomie sieci.
9. Utrzymuj authentication na wewnętrznych panelach admina.
10. Dodaj testy regresji dla znanych wzorców bypassów.

## Główna lekcja

Remediacja SSRF polega na kontroli backend egress.

Backend powinien móc wysyłać requesty tylko do celów, które są jawnie wymagane i bezpieczne.
