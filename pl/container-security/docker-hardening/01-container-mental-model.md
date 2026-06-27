# Mental Model Kontenerów

## Dlaczego to ważne

Docker jest dużo prostszy, gdy podstawowe obiekty są jasne.

Najczęstszy błąd to traktowanie Dockera jak magii. Docker to zestaw mechanizmów dotyczących builda, filesystemu, procesów, sieci i izolacji.

Dla AppSec model mentalny ma znaczenie, bo bezpieczeństwo kontenerów zależy od tego:

- co jest w obrazie,
- co zmienia się w runtime,
- co jest trwałe,
- co jest wystawione na zewnątrz,
- co wolno procesowi.

---

## Image

Docker image to niemodyfikowalny szablon używany do tworzenia kontenerów.

Zwykle zawiera:

- pliki bazowego systemu,
- pliki aplikacji,
- zainstalowane zależności,
- pliki konfiguracyjne,
- metadane komendy startowej,
- metadane portów,
- warstwy filesystemu utworzone podczas builda.

Obraz sam się nie uruchamia. Uruchamia się kontener utworzony z obrazu.

```powershell
docker image ls
```

Pytanie bezpieczeństwa:

```text
Co dokładnie jest w tym obrazie i czy runtime naprawdę tego potrzebuje?
```

Jeżeli obraz zawiera sekrety, narzędzia developerskie, zbędne pakiety albo kod źródłowy, mogą one być dostępne dla uruchomionego procesu.

---

## Container

Kontener to uruchomiona instancja obrazu.

Jest procesem z:

- filesystemem,
- zmiennymi środowiskowymi,
- konfiguracją sieci,
- zamontowanymi wolumenami,
- użytkownikiem procesu,
- ustawieniami runtime.

```powershell
docker ps
```

Kontener można zatrzymać, usunąć i odtworzyć.

Dlatego kontenery powinny być traktowane jako jednorazowe. Ważne dane nie powinny żyć tylko w filesystemie kontenera.

```text
Images zawierają kod aplikacji.
Volumes albo zewnętrzne usługi zawierają mutable data.
```

---

## Image vs container

| Pojęcie | Znaczenie |
|---|---|
| Image | Artefakt builda / szablon |
| Container | Uruchomiony proces utworzony z obrazu |
| Dockerfile | Instrukcje budowania obrazu |
| Docker Compose | Instrukcje uruchamiania usług razem |
| Volume | Trwały storage poza lifecycle jednego kontenera |
| Network | Warstwa komunikacji między kontenerami |

```text
docker build tworzy image
docker run tworzy container z image
docker compose up tworzy wiele containerów/usług
```

---

## Layer

Obraz Dockera jest budowany z warstw.

Każda instrukcja Dockerfile może utworzyć warstwę.

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build
```

Warstwy są ważne, bo:

- Docker może je cache'ować,
- zmiana wcześniejszej warstwy unieważnia późniejsze,
- sekrety skopiowane do warstwy trudno bezpiecznie usunąć,
- duże katalogi zwiększają obraz,
- kolejność warstw wpływa na szybkość builda.

```text
Nie kopiuj sekretów do obrazu po to, żeby później je usuwać.
Użyj build secrets.
```

---

## Build context

Build context to zestaw plików wysyłanych do Dockera podczas builda.

```powershell
docker build -f docker/api/Dockerfile .
```

Końcowa kropka `.` oznacza build context.

Docker może widzieć pliki pod tym katalogiem, chyba że wykluczy je `.dockerignore`.

Dlaczego to ma znaczenie:

- duży context spowalnia build,
- przypadkowe pliki mogą być dostępne podczas builda,
- sekrety mogą zostać przypadkowo skopiowane,
- `COPY . .` może zawierać więcej niż zakładasz.

Typowe wpisy `.dockerignore`:

```text
node_modules
dist
dist-server
.git
.env
*.log
coverage
uploads
*.db
```

```text
Zmniejsz build context. Nie wysyłaj Dockerowi plików, których nie potrzebuje.
```

---

## Base image

Base image to punkt startowy instrukcji `FROM`.

```dockerfile
FROM node:24-bookworm
```

```dockerfile
FROM nginxinc/nginx-unprivileged:stable-alpine
```

Pytania:

- Czy obraz jest zaufany?
- Czy jest utrzymywany?
- Czy jest większy niż potrzeba?
- Czy działa jako root?
- Czy zawiera narzędzia niepotrzebne w runtime?

Build stage może używać większego obrazu, bo potrzebuje narzędzi builda.

Runtime stage powinien być zwykle mniejszy i czystszy.

---

## Dockerfile

Dockerfile odpowiada na pytanie:

```text
Jak zbudować ten obraz?
```

Opisuje pliki, komendy, zależności i metadane startowe jednego obrazu.

```dockerfile
FROM
WORKDIR
COPY
RUN
ENV
USER
EXPOSE
CMD
HEALTHCHECK
```

```text
Dockerfile kontroluje, co trafia do finalnego środowiska runtime.
Złe decyzje w Dockerfile mogą wysłać sekrety, dev tools albo zbędne zależności.
```

---

## Docker Compose

Docker Compose odpowiada na pytanie:

```text
Jak te usługi mają działać razem?
```

Może definiować:

- services,
- images/builds,
- environment variables,
- ports,
- volumes,
- dependencies,
- networks,
- restart policies.

W labie Compose uruchamiał:

```text
api-migrate
api
web
```

```text
Czysty obraz nadal można uruchomić niebezpiecznie, jeżeli konfiguracja runtime/Compose jest słaba.
```

---

## Volume

Docker volume to trwały storage zarządzany przez Dockera.

Istnieje poza lifecycle jednego kontenera.

W labie:

```text
api-data     -> plik bazy SQLite
api-uploads  -> uploadowane pliki
```

Kontenery są jednorazowe. Bez wolumenów usunięcie lub odtworzenie kontenera może usunąć dane.

```text
Mutable data powinny być jawne. Nie polegaj na przypadkowym zapisie wewnątrz filesystemu kontenera.
```

---

## Network

Docker Compose tworzy domyślną sieć dla usług.

Usługi mogą komunikować się po nazwie usługi.

W sieci Dockera:

```text
web -> http://api:3000
```

Z hosta:

```text
browser -> http://localhost:8080
browser -> http://localhost:3000
```

Ważne rozróżnienie:

```text
W kontenerze localhost oznacza ten sam kontener.
Nie oznacza hosta i nie oznacza innej usługi.
```

```text
Publikuj tylko porty wymagające dostępu z hosta. Ruch wewnętrzny może zostać w sieci Dockera.
```

---

## Build-time vs runtime

Build-time:

- instalacja zależności,
- kompilacja TypeScript,
- Vite build,
- generowanie klienta Prisma,
- tworzenie artefaktów.

Runtime:

- start Node API,
- serwowanie statycznych plików przez nginx,
- odczyt environment variables,
- połączenie z bazą,
- zapis uploadów,
- odpowiedzi health check.

```text
Narzędzia build-time nie powinny automatycznie istnieć w runtime.
Jeżeli aplikacja nie potrzebuje narzędzia do działania, nie umieszczaj go w finalnym obrazie.
```

---

## Najważniejszy wniosek

Czysty model Dockera ma cztery warstwy:

```text
Build:
  Dockerfile tworzy image

Image:
  niemodyfikowalny szablon runtime

Run:
  container startuje z image

State:
  volumes/zewnętrzne usługi przechowują dane
```

Dla AppSec każda warstwa ma inne ryzyka:

```text
Build risks:
  wyciek sekretów, niebezpieczna instalacja zależności, niezaufane skrypty

Image risks:
  zbyt dużo narzędzi, podatne pakiety, wyciek kodu źródłowego

Runtime risks:
  root user, zapisywalny filesystem, nadmiarowe capabilities, wystawione porty

State risks:
  utrata danych, słabe uprawnienia, niezaufane uploady, sekrety w plikach
```
