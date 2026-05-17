# Content Discovery i Attack Surface

## 1. Overview

Ta notatka opisuje content discovery oraz mapowanie attack surface.

Aplikacja webowa często jest większa niż to, co widać w menu nawigacyjnym. Niektóre zasoby mogą nie być podlinkowane w UI, ale nadal mogą istnieć i odpowiadać na requesty.

Content discovery pomaga znaleźć takie zasoby.

Attack surface mapping pomaga zrozumieć, gdzie aplikacja może zostać zaatakowana albo nadużyta.

Główna idea:

> Jeśli zasób istnieje i jest osiągalny, musi być poprawnie zabezpieczony.

## 2. Czym jest Content Discovery?

Content Discovery to proces znajdowania zasobów aplikacji, które nie zawsze są widoczne w UI.

Przykłady możliwych do znalezienia zasobów:

- ukryte endpointy,
- katalogi,
- pliki,
- panele admina,
- backupy,
- stare wersje aplikacji,
- trasy API,
- pliki konfiguracyjne,
- katalogi uploadów,
- `robots.txt`,
- `sitemap.xml`,
- komentarze HTML,
- pliki JavaScript,
- linki wewnątrz bundle JavaScript.

Przykładowe ścieżki:

```text
/admin
/api
/backup
/uploads
/dev
/test
/config
```

Content discovery jest przydatne, ponieważ aplikacje często wystawiają więcej zasobów, niż developerzy zakładają.

## 3. Dlaczego UI nie pokazuje całej aplikacji?

UI nie zawsze pokazuje pełną aplikację.

Aplikacja może mieć endpointy, które:

- są wywoływane tylko przez JavaScript,
- nie są podlinkowane w menu,
- są przeznaczone dla adminów,
- są stare, ale nadal wdrożone,
- były używane do testów,
- zostały przypadkowo zostawione na produkcji,
- są używane przez aplikacje mobilne,
- są używane przez narzędzia wewnętrzne,
- są częścią API,
- są dostępne przez przewidywalne nazwy.

Brak linka w UI nie oznacza, że zasób jest prywatny.

Na przykład menu może nie pokazywać:

```text
/admin
```

ale URL może nadal istnieć.

## 4. Ukryte URL-e nie są zabezpieczeniem

Ukryty URL nie jest prawdziwą kontrolą bezpieczeństwa.

Przykład:

```text
/admin
/backup
/dev
/api/users
/uploads
```

Nawet jeśli nie ma widocznego linka do tych ścieżek, mogą one zostać znalezione przez:

- `robots.txt`,
- `sitemap.xml`,
- DevTools,
- Burp history,
- pliki JavaScript,
- komentarze HTML,
- wordlisty,
- wyszukiwarki,
- zarchiwizowane strony,
- zgadywanie popularnych nazw.

To łączy się z pojęciem security by obscurity.

Security by obscurity oznacza poleganie na tym, że coś jest ukryte, zamiast poprawnie zabezpieczone.

Ukrycie może zmniejszyć przypadkowe znalezienie zasobu, ale to nie wystarczy. Wrażliwe zasoby nadal potrzebują poprawnej kontroli dostępu po stronie serwera.

## 5. robots.txt

`robots.txt` mówi crawlerom wyszukiwarek, których ścieżek nie powinny indeksować.

Przykład:

```text
User-agent: *
Disallow: /admin
Disallow: /backup
```

Z perspektywy AppSec ten plik może być ciekawy, ponieważ może ujawniać wrażliwe albo ukryte ścieżki.

Ważny punkt:

> robots.txt niczego nie zabezpiecza. On tylko prosi crawlery, żeby nie indeksowały danych ścieżek.

Jeśli `/admin` znajduje się w `robots.txt`, nie powinienem zakładać, że jest bezpieczny. Powinienem sprawdzić, jak serwer odpowiada na request do tej ścieżki.

Możliwe bezpieczne zachowania:

- redirect do logowania,
- `401 Unauthorized`,
- `403 Forbidden`,
- czasem `404 Not Found`,
- poprawna kontrola dostępu po authentication.

## 6. sitemap.xml

`sitemap.xml` może zawierać listę URL-i aplikacji.

Może ujawnić:

- strony niewidoczne w menu,
- stare strony,
- nietypowe ścieżki,
- sekcje aplikacji,
- strony testowe,
- typy contentu,
- endpointy lub publiczne routes.

Przykład:

```text
/sitemap.xml
```

Sitemap files są przydatne podczas mapowania aplikacji, bo dają szybki podgląd znanych URL-i.

Jednak to, że URL znajduje się w sitemapie, nie oznacza, że jest bezpieczny. Każdy URL nadal potrzebuje poprawnej kontroli dostępu i bezpiecznego zachowania.

## 7. Favicon Fingerprinting

Favicon czasem może pomóc rozpoznać technologię, framework, panel admina, CMS albo self-hosted application.

Nazywa się to favicon fingerprinting.

Na przykład, jeśli aplikacja używa domyślnego faviconu znanego narzędzia, może to zdradzić, jaka technologia działa pod spodem.

Może to pomóc w dalszym sprawdzaniu, ponieważ znane technologie mogą mieć:

- domyślne ścieżki,
- domyślne panele admina,
- znane pliki konfiguracyjne,
- podatności zależne od wersji,
- typowe błędy konfiguracji.

Favicon fingerprinting sam w sobie nie jest dowodem podatności. To tylko wskazówka.

## 8. Ręczne mapowanie aplikacji

Ręczne mapowanie oznacza przechodzenie po aplikacji i zapisywanie, jak działa.

Podczas manual mapping powinienem szukać:

- stron,
- formularzy,
- przycisków,
- linków,
- requestów API,
- parametrów,
- cookies,
- headers,
- redirectów,
- funkcji uploadu,
- logowania i wylogowania,
- resetu hasła,
- funkcji admina,
- nietypowych odpowiedzi serwera.

Manual mapping jest ważne, bo daje kontekst biznesowy.

Na przykład powinienem rozumieć:

- Co robi aplikacja?
- Jakie dane przetwarza?
- Gdzie przyjmuje input od użytkownika?
- Które akcje zmieniają dane?
- Które funkcje wymagają authentication?
- Które funkcje wymagają specjalnych uprawnień?
- Które requesty zawierają ID użytkownika albo zasobu?

Narzędzia automatyczne mogą znaleźć ścieżki, ale ręczne mapowanie pomaga zrozumieć znaczenie i ryzyko.

## 9. Automated Discovery

Automated discovery używa narzędzi i wordlist do szukania popularnych ścieżek.

Przykłady narzędzi:

- Gobuster,
- ffuf,
- dirsearch,
- Burp Intruder,
- Burp Content Discovery features.

Przykładowe ścieżki testowane przez wordlistę:

```text
/admin
/api
/backup
/uploads
/dev
/test
/config
```

Automatyczne narzędzia są przydatne, ale nie są magiczne.

Znajdują tylko to, czego spróbują.

Wyniki zależą od:

- wordlisty,
- konfiguracji targetu,
- status codes,
- redirectów,
- rate limits,
- authentication,
- rozszerzeń plików,
- naming conventions.

Dobre podejście łączy automated discovery z ręcznym myśleniem.

## 10. Czym jest Attack Surface?

Attack surface oznacza wszystkie miejsca, w których aplikacja może zostać zaatakowana, nadużyta albo użyta w niezamierzony sposób.

Przykłady attack surface:

- login,
- registration,
- password reset,
- upload plików,
- formularze,
- API endpoints,
- parametry URL,
- cookies,
- headers,
- panele admina,
- publiczne katalogi,
- integracje z zewnętrznymi serwisami,
- stare endpointy,
- subdomeny,
- webhooks,
- wyszukiwarki,
- płatności lub checkout.

Im większa attack surface, tym więcej miejsc wymaga:

- walidacji,
- authentication,
- authorization,
- bezpiecznej konfiguracji,
- logowania zdarzeń,
- obsługi błędów,
- rate limiting.

Attack surface mapping pomaga zdecydować, gdzie skupić testowanie.

## 11. Ciekawe ścieżki do sprawdzenia

Niektóre ścieżki są szczególnie ciekawe podczas testowania AppSec.

### `/api`

API może wystawiać dane albo akcje używane przez frontend lub aplikację mobilną.

Potencjalne ryzyka:

- IDOR,
- BOLA,
- Broken Access Control,
- excessive data exposure,
- brak authentication,
- brak authorization,
- słaba walidacja inputu.

### `/admin`

Panele admina są wrażliwe, ponieważ często umożliwiają wykonywanie akcji uprzywilejowanych.

Poprawne zachowanie serwera może obejmować:

- redirect do logowania,
- `401 Unauthorized`, jeśli użytkownik nie jest zalogowany,
- `403 Forbidden`, jeśli użytkownik jest zalogowany, ale nie jest adminem,
- poprawne sprawdzanie authorization,
- czasem `404 Not Found`, żeby nie ujawniać zasobu.

### `/backup`

Ścieżki backupowe mogą być groźne, jeśli są publicznie dostępne.

Możliwe pliki:

```text
backup.zip
db.sql
config.bak
site.old
database.dump
```

Potencjalne ryzyka:

- wyciek kodu źródłowego,
- wyciek dumpa bazy danych,
- wyciek credentials,
- ujawnienie konfiguracji,
- stary podatny kod.

### `/uploads`

Katalogi uploadów są ciekawe, bo funkcje uploadu plików mogą prowadzić do poważnych podatności.

Potencjalne ryzyka:

- publiczny dostęp do uploadowanych plików,
- niebezpieczne typy plików,
- słaba walidacja plików,
- path traversal,
- stored XSS,
- upload malware,
- nadpisywanie plików.

### `/dev` i `/test`

Ścieżki developerskie lub testowe mogą ujawniać niedokończone albo mniej bezpieczne funkcje.

Potencjalne ryzyka:

- debug information,
- konta testowe,
- verbose errors,
- narzędzia wewnętrzne,
- słabsza kontrola dostępu,
- tymczasowe endpointy zostawione na produkcji.

## 12. Key Takeaways

- Aplikacja to więcej niż widoczne linki.
- Ukryty URL nie jest prawdziwym zabezpieczeniem.
- `robots.txt` i `sitemap.xml` mogą ujawniać ciekawe ścieżki.
- Favicon fingerprinting może dać wskazówki technologiczne.
- Manual mapping pomaga zrozumieć kontekst biznesowy.
- Automated discovery pomaga znaleźć popularne ścieżki.
- Narzędzia są przydatne, ale zależą od wordlist i konfiguracji.
- Attack surface obejmuje każde miejsce, gdzie istnieje input, access lub actions.
- Panele admina, API, backupy i uploady są szczególnie ciekawe.
- Wrażliwe zasoby muszą być chronione przez poprawne authentication i authorization.

## 13. Practical Discovery Checklist

Podczas content discovery powinienem sprawdzić:

- Czy istnieje `robots.txt`?
- Czy istnieje `sitemap.xml`?
- Czy w plikach JavaScript są ciekawe ścieżki?
- Czy w HTML są komentarze?
- Czy w DevTools albo Burpie widać requesty API?
- Czy są ukryte formularze albo parametry?
- Czy są ścieżki związane z adminem?
- Czy są backup files?
- Czy są katalogi uploadów?
- Czy stare albo testowe ścieżki nadal są dostępne?
- Jakie status codes zwracają ciekawe ścieżki?
- Czy serwer wymaga authentication?
- Czy serwer sprawdza authorization?
- Czy response ujawnia wrażliwe dane?

Praktyczny mindset:

> Znalezienie ukrytej ścieżki nie jest podatnością samo w sobie. Podatność pojawia się wtedy, gdy ścieżka ujawnia dane, pozwala na nieautoryzowane akcje albo jest źle skonfigurowana.
