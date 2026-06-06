# OS Command Injection - Podsumowanie labów

## Porównanie

| Obszar | Prosty lab | Blind output-redirection lab |
|---|---|---|
| Funkcja | product stock checker | feedback submission |
| Endpoint | `POST /product/stock` | `POST /feedback/submit` |
| Potwierdzony parametr | `storeId` | `email` |
| Typ injection | direct/verbose | blind |
| Pierwszy użyteczny dowód | output `whoami` w odpowiedzi | proporcjonalny timing delay |
| Finalny kanał outputu | ta sama odpowiedź HTTP | plik plus osobny request `/image` |
| Główny side effect | stdout zwrócony bezpośrednio | stdout przekierowany do zapisywalnego pliku |
| Główna trudność | identyfikacja direct outputu | oddzielenie błędów od dowodu i znalezienie side channel |

## Wspólny model mentalny

```text
kontrolowany input
-> budowa komendy w backendzie
-> interpretacja przez shell
-> zmienione OS execution
-> direct output albo side effect
```

## Lekcje o dowodach

### Prosty lab

Odpowiedź zawierała normalną wartość stock i username z `whoami`. Direct output wystarczył do potwierdzenia execution.

### Blind lab

Generyczne `500` nie wystarczyło. Mocny dowód dały:

1. milisekundy dla normalnych requestów,
2. około 4,1 sekundy dla `ping -c 5`,
3. około 9,3 sekundy dla `ping -c 10`,
4. plik zawierający output `whoami`,
5. pobranie tego pliku przez `/image`.

## Główne korekty

- Wybieraj parametry przez data-flow reasoning, nie tylko założenia o walidacji.
- Oddzielaj fakty od zgadywania oryginalnej komendy backendowej.
- Błąd odpowiedzi nie dowodzi command execution.
- Timing powinien być powtarzalny i proporcjonalny.
- Separator zamykający może być potrzebny do izolacji wstrzykniętej komendy.
- `>` przekierowuje stdout do pliku.
- Endpoint pobierania może być side channel, a nie podatnym sinkiem.
- Sanitizacja nie jest główną strategią remediacji.

## Takeaways dla developera

- Unikaj komend systemowych, gdy dostępne są biblioteki albo API aplikacyjne.
- Unikaj shella, gdy process execution jest konieczne.
- Przekazuj stały executable i osobne zwalidowane argumenty.
- Rozważ argument injection nawet bez interpretacji shella.
- Stosuj least privilege, ograniczenia filesystemu, timeouty, limity outputu, logging i monitoring.
- Dodaj testy regresji sprawdzające zachowanie i implementację, nie tylko walidację inputu.

## Odpowiedzi interview-style

### Czym jest OS Command Injection?

To podatność, w której niezaufany input trafia do komendy albo process execution i zmienia to, co wykonuje system operacyjny.

### Jaka jest różnica między direct i blind command injection?

Direct injection zwraca output komendy w podatnej odpowiedzi. Blind injection wykonuje się bez zwracania stdout, więc dowody pochodzą z timingu, plików albo outbound interactions.

### Jaki był root cause w feedback labie?

Backend pozwolił, aby kontrolowana przez użytkownika wartość email wpłynęła na string komendy interpretowany przez shell.

### Dlaczego odpowiedź `500` nie wystarczyła?

Pokazała, że przetwarzanie się nie powiodło, ale nie dowodziła wykonania dodatkowej komendy. Wykonanie potwierdziły powtarzalny timing i output w pliku.

### Jaka jest główna poprawka?

Usuń komendę shella tam, gdzie to możliwe. W przeciwnym razie użyj stałego executable i osobnych zwalidowanych argumentów z wyłączonym shellem, a potem dodaj allowlisty i defence-in-depth controls.
