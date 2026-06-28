# Model myślenia o bezpieczeństwie CI/CD

## Co oznacza bezpieczeństwo CI/CD

Bezpieczeństwo CI/CD chroni drogę między zmianą w kodzie a zaufanym buildem albo merge.

Pipeline nie jest tylko pomocnikiem. To zaufany system, który może uruchamiać kod, instalować paczki, wykonywać skrypty, budować kontenery, a czasem korzystać z tokenów albo sekretów.

To oznacza, że słaby pipeline może stać się sposobem na ominięcie normalnych kontroli bezpieczeństwa aplikacji.

## Co może pójść źle

Pipeline CI/CD może być źle skonfigurowany albo nadużyty na kilka sposobów:

```text
token workflowa ma zbyt szerokie uprawnienia
niezaufany kod uruchamia się z dostępem do sekretów
złośliwy skrypt z zależności uruchamia się podczas instalacji
kontener się buduje, ale nie startuje poprawnie
znana podatna zależność jest ignorowana
wyniki SAST nigdy nie są sprawdzane
ochronę gałęzi da się ominąć
logi ujawniają wrażliwe wartości
podwójne uruchomienia CI robią szum i ukrywają realne błędy
```

## Praktyczny model

W sprincie użyłem takiego modelu warstwowego:

```text
1. Sprawdź kod
   format, lint, typecheck, testy, build, walidacja Prisma

2. Sprawdź działanie kontenera
   Docker build, Docker run, API health check, logi przy błędzie

3. Sprawdź zależności
   npm audit JSON i skrypt z polityką

4. Sprawdź wzorce w kodzie
   CodeQL dla JavaScript/TypeScript

5. Chroń przepływ zmian w repo
   brak direct push do master, zmiany przez pull request, wymagane checki

6. Utrzymaj porządek w pipeline
   mniej duplikatów CI, ignorowane pliki lokalne, brak zbędnych sekretów
```

## Dlaczego najmniejsze uprawnienia są ważne

Zwykły job walidacyjny najczęściej musi tylko przeczytać repozytorium. Nie powinien dostawać uprawnień zapisu bez konkretnego powodu.

Bezpieczniejszy domyślny wariant:

```yaml
permissions:
  contents: read
```

To zmniejsza skutki błędu albo kompromitacji wewnątrz joba.

## Dlaczego timeout i concurrency mają znaczenie

Timeout zatrzymuje joby, które wiszą za długo.

Concurrency anuluje stare uruchomienia, gdy pojawi się nowszy commit.

Te ustawienia poprawiają niezawodność, zmniejszają marnowanie zasobów i ułatwiają czytanie wyników pull requesta.

## Dlaczego trzeba sprawdzać działanie kontenera

Poprawny build nie oznacza jeszcze, że aplikacja uruchamia się poprawnie.

Dla API w Dockerze pipeline powinien też sprawdzić:

```text
czy obraz się buduje
czy kontener startuje
czy wymagane zmienne środowiskowe są przekazane
czy API nasłuchuje na oczekiwanym porcie
czy health endpoint odpowiada
czy logi są dostępne przy błędzie
czy cleanup działa także po błędzie
```

## Dlaczego skanery nadal wymagają triage

Skanery zależności i narzędzia SAST są przydatne, ale nie zastępują oceny inżynierskiej.

Dobry triage pyta:

```text
Czy zależność jest direct czy transitive?
Czy jest dostępna poprawka?
Czy ten kod działa w runtime, czy tylko w development?
Czy problem da się realnie wykorzystać w tym projekcie?
Czy naprawiamy teraz, czy opisujemy tymczasową akceptację ryzyka?
Kiedy zaakceptowane ryzyko wygasa?
```

## Końcowy model myślenia

Dobry setup CI/CD powinien ułatwiać bezpieczną ścieżkę i pokazywać ryzykowne miejsca.

Celem nie jest dodawanie narzędzi dla samego dodawania. Celem są sensowne checki, które dają zaufanie przed merge.
