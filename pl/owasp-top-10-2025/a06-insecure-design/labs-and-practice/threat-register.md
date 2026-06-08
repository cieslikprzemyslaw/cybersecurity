# Fikcyjna Event Booking Platform - Threat Register

Ten rejestr został wygenerowany z threat metadata zapisanych w dostarczonym modelu draw.io.

Pola zachowują kolejność alfabetyczną A-Z zgodną z properties w draw.io:

```text
Dotknięty element
Opis
Mitygacja
Severity
Status
STRIDE
Uzasadnienie STRIDE
Threat ID
Tytuł
Weryfikacja
```

`Open` oznacza, że kontrola w fikcyjnym modelu nie została zaimplementowana i zweryfikowana. Nie jest to dowód podatności w realnym systemie produkcyjnym.

## T-001 - Manipulacja żądaniami uwierzytelniania i operacji na rolach

### Dotknięty element

Żądania uwierzytelniania, resetu hasła i operacji na rolach na przepływie API Gateway → Auth Service

### Opis

Atakujący modyfikuje wartości dotyczące tożsamości, roli, resetu hasła lub operacji na koncie wysyłane do Auth Service. Jeżeli wartości kontrolowane przez klienta są traktowane jako autorytatywne, atakujący może wpłynąć na inne konto albo operację uprzywilejowaną.

### Mitygacja

Traktować wszystkie wartości tożsamości, roli i resetu pochodzące od klienta jako niezaufane. Ustalać tożsamość wykonującego operację na podstawie zweryfikowanej sesji lub podpisanego tokenu, pobierać role z zaufanych danych serwerowych, weryfikować tokeny resetu i egzekwować autoryzację dla każdej wrażliwej operacji.

### Severity

High

### Status

Open

### STRIDE

Tampering

### Uzasadnienie STRIDE

Atakujący zmienia dane istotne dla bezpieczeństwa podczas przejścia przez granicę zaufania klient/serwer.

### Threat ID

T-001

### Tytuł

Manipulacja żądaniami uwierzytelniania i operacji na rolach

### Weryfikacja

Zmienić wartości user ID, roli, adresu e-mail i tokenu resetu hasła. Backend musi odrzucić nieautoryzowane lub nieprawidłowe żądania, nie zmienić stanu i zapisać odrzuconą próbę.

---

## T-002 - Eskalacja uprawnień przez zaufanie roli przesłanej przez klienta

### Dotknięty element

Decyzje Auth Service dotyczące ról i uprawnień

### Opis

Auth Service akceptuje lub nadaje podwyższone uprawnienia na podstawie roli, user ID lub wartości konta kontrolowanej przez klienta zamiast zaufanych danych autoryzacyjnych po stronie serwera.

### Mitygacja

Ignorować role i uprawnienia przesyłane przez klienta. Pobierać dane autoryzacyjne z zaufanych źródeł serwerowych, domyślnie odmawiać dostępu, weryfikować integralność tokenu i wymagać ponownego uwierzytelnienia przy zmianach ról wysokiego ryzyka. Serwisy docelowe muszą egzekwować autoryzację dla każdej wrażliwej akcji.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### Uzasadnienie STRIDE

Uwierzytelniony klient może uzyskać możliwości administratora lub inne uprawnienia wykraczające poza jego dozwolony zakres.

### Threat ID

T-002

### Tytuł

Eskalacja uprawnień przez zaufanie roli przesłanej przez klienta

### Weryfikacja

Zalogować się jako zwykły klient i przesłać rolę administratora albo bezpośrednio wywołać operacje zarządzania rolami i endpointy administracyjne. Backend musi zwrócić 403 i nie zmienić stanu.

---

## T-003 - Przejęcie konta przez niebezpieczne tokeny resetu hasła

### Dotknięty element

Workflow resetu hasła w Auth Service

### Opis

Atakujący pozyskuje, przewiduje, ponownie wykorzystuje lub nadużywa tokenu resetu hasła, aby ustawić nowe hasło dla konta innego użytkownika.

### Mitygacja

Generować kryptograficznie losowe tokeny resetu o wysokiej entropii. Powiązać każdy token z jednym kontem i jedną operacją resetu, używać krótkiego czasu ważności, unieważniać token natychmiast po poprawnym użyciu, ograniczać liczbę prób i zapobiegać wyciekom tokenów przez logi, analitykę lub żądania do podmiotów trzecich.

### Severity

High

### Status

Open

### STRIDE

Spoofing

### Uzasadnienie STRIDE

Skuteczny atak na token resetu pozwala podszyć się pod prawowitego właściciela konta.

### Threat ID

T-003

### Tytuł

Przejęcie konta przez niebezpieczne tokeny resetu hasła

### Weryfikacja

Ponownie użyć wykorzystanego tokenu, przesłać token wygasły lub zmodyfikowany oraz spróbować użyć tokenu z innym kontem. Każda próba musi zostać odrzucona bez zmiany hasła.

---

## T-004 - Enumeracja kont przez niespójne odpowiedzi uwierzytelniania

### Dotknięty element

Odpowiedzi uwierzytelniania i odzyskiwania konta na przepływie Auth Service → API Gateway

### Opis

Atakujący porównuje odpowiedzi dla istniejących i nieistniejących nazw użytkowników lub adresów e-mail podczas logowania, rejestracji albo odzyskiwania hasła. Różnice w komunikacie, statusie HTTP, długości odpowiedzi, przekierowaniu lub czasie mogą ujawnić, które konta istnieją.

### Mitygacja

Zwracać spójne odpowiedzi dla istniejących i nieistniejących kont, ograniczać obserwowalne różnice czasowe oraz stosować rate limiting, monitoring i wykrywanie nadużyć przy powtarzanych próbach uwierzytelniania i odzyskiwania konta.

### Severity

Medium

### Status

Open

### STRIDE

Information Disclosure

### Uzasadnienie STRIDE

Aplikacja ujawnia informację o istnieniu konta osobie, która nie powinna jej otrzymać.

### Threat ID

T-004

### Tytuł

Enumeracja kont przez niespójne odpowiedzi uwierzytelniania

### Weryfikacja

Wysłać równoważne żądania logowania, rejestracji i odzyskiwania hasła dla istniejącego i nieistniejącego konta. Porównać komunikat, status, długość, przekierowanie i czas. Odpowiedzi nie mogą wiarygodnie ujawniać, czy konto istnieje.

---

## T-005 - Podszycie się pod konto przez nieunieważnione sesje

### Dotknięty element

Cykl życia sesji i tokenów w Auth Service

### Opis

Wcześniej wydana lub skradziona sesja pozostaje ważna po wylogowaniu, resecie hasła, zmianie hasła, zablokowaniu konta albo zmianie roli istotnej dla bezpieczeństwa. Osoba posiadająca token sesji może nadal działać jako prawowity użytkownik.

### Mitygacja

Zdefiniować jednoznaczną politykę unieważniania sesji. Unieważniać odpowiednie sesje po wylogowaniu, resecie, zmianie hasła i zablokowaniu konta; rotować identyfikatory sesji po uwierzytelnieniu i zmianach uprawnień; weryfikować wygaśnięcie, stan konta i revocation state; umożliwić użytkownikowi unieważnienie innych aktywnych sesji.

### Severity

High

### Status

Open

### STRIDE

Spoofing

### Uzasadnienie STRIDE

Nieaktualna lub skradziona sesja pozwala atakującemu podszyć się pod uwierzytelnionego właściciela konta.

### Threat ID

T-005

### Tytuł

Podszycie się pod konto przez nieunieważnione sesje

### Weryfikacja

Zalogować się do tego samego konta w dwóch przeglądarkach. W pierwszej wykonać wylogowanie, reset hasła, zmianę hasła, blokadę konta i zmianę roli. Następnie sprawdzić starą sesję w drugiej przeglądarce; musi zostać odrzucona zgodnie ze zdefiniowaną polityką.

---

## T-006 - Nieautoryzowane zarządzanie wydarzeniem innego organizatora

### Dotknięty element

Operacje zarządzania wydarzeniami w Event Service

### Opis

Uwierzytelniony organizer zmienia event ID i próbuje wyświetlić, zaktualizować, opublikować lub usunąć wydarzenie należące do innego organizatora. Jeżeli serwis sprawdza jedynie, czy użytkownik jest zalogowany, organizer może działać poza swoim dozwolonym zakresem.

### Mitygacja

Ustalać tożsamość wykonującego operację z poprawnej sesji lub podpisanego tokenu. Egzekwować autoryzację obiektową dla każdego odczytu, update, publikacji i usunięcia wydarzenia. Weryfikować ownership lub jawnie nadane uprawnienie do zarządzania, domyślnie odmawiać dostępu i nie polegać na ukrytych kontrolkach frontendu.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### Uzasadnienie STRIDE

Organizer próbuje rozszerzyć prawidłowe uprawnienia do własnych wydarzeń na wydarzenia innego organizatora.

### Threat ID

T-006

### Tytuł

Nieautoryzowane zarządzanie wydarzeniem innego organizatora

### Weryfikacja

Utworzyć wydarzenia na kontach organizatorów A i B. Jako A podmienić event ID na ID wydarzenia B w żądaniach odczytu, update, publikacji i usunięcia. Backend musi odrzucić wszystkie operacje, nie ujawnić prywatnych danych i nie zmienić stanu.

---

## T-007 - Ujawnienie prywatnych wydarzeń nieautoryzowanym użytkownikom

### Dotknięty element

Listingi, wyszukiwanie i odpowiedzi szczegółów wydarzenia w Event Service

### Opis

Prywatne lub ograniczone wydarzenie pojawia się w publicznych wynikach wyszukiwania, listingach lub odpowiedziach API albo jego szczegóły można pobrać bezpośrednio jako nieautoryzowany użytkownik znający lub zgadujący event ID.

### Mitygacja

Egzekwować zasady widoczności i dostępu w Event Service dla każdego listingu, wyszukiwania i żądania szczegółów. Filtrować nieautoryzowane rekordy przed budowaniem odpowiedzi, zapewnić respektowanie zmian widoczności przez cache i indeksy wyszukiwania oraz zwracać wyłącznie minimalny dozwolony zestaw pól.

### Severity

High

### Status

Open

### STRIDE

Information Disclosure

### Uzasadnienie STRIDE

Prywatne informacje o wydarzeniu są zwracane użytkownikowi, który nie ma prawa ich otrzymać.

### Threat ID

T-007

### Tytuł

Ujawnienie prywatnych wydarzeń nieautoryzowanym użytkownikom

### Weryfikacja

Utworzyć prywatne wydarzenie dostępne tylko dla wybranego użytkownika lub grupy. Jako nieautoryzowany użytkownik i niezalogowany gość sprawdzić listingi, wyszukiwanie, bezpośredni endpoint szczegółów oraz zmodyfikowane event ID. Wydarzenie i jego prywatne metadane nie mogą zostać zwrócone.

---

## T-008 - Nieprawidłowy lub nieautoryzowany stan wydarzenia przez zmanipulowane dane

### Dotknięty element

Operacje tworzenia i aktualizacji wydarzeń w Event Service

### Opis

Organizer manipuluje kontrolowanymi przez klienta wartościami, takimi jak capacity, owner ID, status publikacji lub daty wydarzenia. Bez serwerowych invariants serwis może zapisać nieprawidłowy albo nieautoryzowany stan wydarzenia.

### Mitygacja

Walidować dane wydarzenia po stronie serwera. Egzekwować limity capacity i prawidłowe relacje między datami, ustalać ownership z zaufanych danych tożsamości, modelować dozwolone przejścia statusu publikacji oraz odrzucać nieznane, nieprawidłowe i nieautoryzowane zmiany przed zapisem do bazy.

### Severity

High

### Status

Open

### STRIDE

Tampering

### Uzasadnienie STRIDE

Atakujący zmienia autorytatywne dane wydarzenia i może doprowadzić do zapisania nieprawidłowego lub nieautoryzowanego stanu.

### Threat ID

T-008

### Tytuł

Nieprawidłowy lub nieautoryzowany stan wydarzenia przez zmanipulowane dane

### Weryfikacja

Przesłać ujemne, zerowe i bardzo duże wartości capacity, datę zakończenia wcześniejszą od rozpoczęcia, zmodyfikowany owner ID oraz niedozwolone przejścia publikacji. Backend musi zwrócić kontrolowany błąd i nie zmienić danych w bazie.

---

## T-009 - Nieautoryzowany dostęp do rezerwacji innego klienta

### Dotknięty element

Operacje rezerwacji, anulowania i biletów w Booking Service

### Opis

Uwierzytelniony klient zmienia booking ID i próbuje wyświetlić, zaktualizować, anulować albo pobrać bilet należący do innego klienta. Jeżeli sprawdzane jest tylko uwierzytelnienie, klient może działać poza swoim dozwolonym zakresem.

### Mitygacja

Ustalać tożsamość z poprawnej sesji lub podpisanego tokenu. Egzekwować autoryzację obiektową dla każdego odczytu, update, anulowania i pobrania biletu. Weryfikować ownership albo jawne uprawnienie administracyjne i zwracać tylko minimalny zakres dozwolonych danych.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### Uzasadnienie STRIDE

Klient próbuje rozszerzyć uprawnienia z własnych rezerwacji na rezerwację należącą do innego użytkownika.

### Threat ID

T-009

### Tytuł

Nieautoryzowany dostęp do rezerwacji innego klienta

### Weryfikacja

Utworzyć rezerwacje na kontach klientów A i B. Jako A podmienić booking ID na ID rezerwacji B w żądaniach odczytu, update, anulowania i pobrania biletu. Backend musi odrzucić każdą próbę, nie zwrócić danych rezerwacji i nie zmienić stanu.

---

## T-010 - Nieprawidłowe przejścia stanu rezerwacji i biletu

### Dotknięty element

Cykl życia rezerwacji i biletu w Booking Service

### Opis

Klient modyfikuje żądania, pomija kroki, zmienia ich kolejność lub ponawia operacje, aby potwierdzić oczekującą rezerwację, uzyskać ważny bilet przed płatnością, ponownie aktywować anulowaną rezerwację albo nadal używać biletu po anulowaniu lub refundzie.

### Mitygacja

Wdrożyć jawną serwerową maszynę stanów dla rezerwacji i biletów. Zezwalać wyłącznie na zdefiniowane przejścia, ustalać stan płatności z zaufanych danych Payment Service, sprawdzać bieżący stan przed każdą zmianą i atomowo unieważniać bilety po anulowaniu lub refundzie.

### Severity

High

### Status

Open

### STRIDE

Tampering

### Uzasadnienie STRIDE

Atakujący zmienia autorytatywny stan rezerwacji lub biletu albo wymusza stan niedozwolony przez reguły biznesowe.

### Threat ID

T-010

### Tytuł

Nieprawidłowe przejścia stanu rezerwacji i biletu

### Weryfikacja

Spróbować pominąć płatność, zmienić kolejność potwierdzenia i płatności, ponowić żądania potwierdzenia, reaktywować anulowaną rezerwację oraz użyć biletu po anulowaniu lub refundzie. Każde niedozwolone przejście musi zostać odrzucone bez zmiany stanu.

---

## T-011 - Overbooking przez równoległe żądania rezerwacji

### Dotknięty element

Booking Service, Event Service i przepływ danych capacity rezerwacji

### Opis

Wiele jednoczesnych żądań rezerwuje te same ostatnie miejsca, ponieważ dostępność jest sprawdzana i aktualizowana nieatomowo. System może potwierdzić więcej biletów, niż pozwala capacity wydarzenia.

### Mitygacja

Egzekwować invariant capacity w transakcji lub przy użyciu innej atomowej kontroli współbieżności. Stosować constraints bazy danych, locking lub compare-and-set zależnie od potrzeb. Zapewnić idempotentne tworzenie rezerwacji i ponownie sprawdzać dostępność na autorytatywnej granicy zapisu.

### Severity

High

### Status

Open

### STRIDE

Tampering

### Uzasadnienie STRIDE

Równoległe żądania powodują niedozwolone zmiany stanu rezerwacji i capacity, naruszając integralność puli miejsc.

### Threat ID

T-011

### Tytuł

Overbooking przez równoległe żądania rezerwacji

### Weryfikacja

Utworzyć wydarzenie z jednym wolnym miejscem i wysłać wiele żądań rezerwacji równolegle z oddzielnych sesji. Maksymalnie jedno żądanie może się udać, capacity nie może spaść poniżej zera i nie mogą powstać zduplikowane potwierdzone bilety.

---

## T-012 - Manipulacja kwotą płatności przez niezaufane dane rezerwacji

### Dotknięty element

Żądania tworzenia płatności i refundu na przepływie Booking Service → Payment Service

### Opis

Klient manipuluje kwotą, walutą, booking ID lub wartością refundu. Jeżeli Payment Service ufa wartościom pochodzącym od klienta albo niezweryfikowanego upstream requestu, może zostać wykonana nieprawidłowa płatność lub refund.

### Mitygacja

Obliczać kwoty płatności i refundów na podstawie zaufanych serwerowych danych rezerwacji, cen i polityk. Powiązać rekord płatności z uwierzytelnionym klientem i rezerwacją, walidować walutę oraz stan i ponownie wyliczać autorytatywną kwotę przed utworzeniem płatności lub refundu.

### Severity

High

### Status

Open

### STRIDE

Tampering

### Uzasadnienie STRIDE

Atakujący zmienia wartości finansowe używane przez workflow płatności.

### Threat ID

T-012

### Tytuł

Manipulacja kwotą płatności przez niezaufane dane rezerwacji

### Weryfikacja

Zmienić kwotę, walutę, booking ID i wartość refundu w przechwyconych żądaniach. Payment Service musi zignorować lub odrzucić zmanipulowane wartości i użyć wyłącznie kwoty obliczonej przez serwer.

---

## T-013 - Fałszywe potwierdzenie płatności przez podrobiony lub replayowany webhook

### Dotknięty element

Przepływ webhooka External Payment Provider → Payment Service

### Opis

Atakujący wysyła sfałszowany callback statusu płatności albo ponawia wcześniej prawidłowy webhook. Jeżeli autentyczność i świeżość nie są sprawdzane, rezerwacja może zostać oznaczona jako opłacona lub zrefundowana bez prawidłowego zdarzenia od providera.

### Mitygacja

Weryfikować podpis providera na surowym payloadzie, timestamp i typ zdarzenia, odrzucać stare wiadomości, zapisywać provider event ID i przetwarzać każde zdarzenie tylko raz. Przed zmianą stanu powiązać event z oczekiwaną płatnością, rezerwacją, kwotą i walutą.

### Severity

High

### Status

Open

### STRIDE

Spoofing

### Uzasadnienie STRIDE

Atakujący podszywa się pod zewnętrznego providera płatności, przesyłając fałszywy callback.

### Threat ID

T-013

### Tytuł

Fałszywe potwierdzenie płatności przez podrobiony lub replayowany webhook

### Weryfikacja

Wysłać webhooki bez podpisu, z nieprawidłowym podpisem, zmodyfikowane, wygasłe i ponownie użyte. Stan płatności, refundu, rezerwacji ani biletu nie może się zmienić, jeśli zdarzenie nie jest autentyczne, aktualne, oczekiwane i wcześniej nieprzetworzone.

---

## T-014 - Podwójna płatność lub refund przez powtarzane operacje

### Dotknięty element

Operacje płatności, refundów i Payment Database w Payment Service

### Opis

Retry, ponowione żądania lub operacje równoległe tworzą tę samą płatność albo refund więcej niż raz, ponieważ workflow nie ma idempotencji i walidacji stanu.

### Mitygacja

Wymagać unikalnych idempotency keys przy tworzeniu płatności i refundów, zapisywać je z unique constraints, modelować dozwolone przejścia stanów finansowych i sprawić, aby identyczne powtórzone żądania zwracały istniejący wynik bez ponownego wykonania operacji.

### Severity

High

### Status

Open

### STRIDE

Tampering

### Uzasadnienie STRIDE

Powtarzane żądania powodują zduplikowane lub niespójne zmiany w autorytatywnych rekordach finansowych.

### Threat ID

T-014

### Tytuł

Podwójna płatność lub refund przez powtarzane operacje

### Weryfikacja

Powtórzyć i równolegle wysłać identyczne żądania płatności i refundu dla tej samej operacji biznesowej. Może zostać wykonana dokładnie jedna operacja finansowa, a wszystkie retry muszą zwrócić spójny istniejący wynik.

---

## T-015 - Wrażliwe dane rezerwacji lub płatności wysłane do złego odbiorcy

### Dotknięty element

Obsługa odbiorcy i danych template w Notification Service

### Opis

Adres odbiorcy, odwołanie do rezerwacji lub zmienna template pochodzi z niezaufanego inputu albo nieaktualnego mapowania, przez co informacje o bilecie, rezerwacji lub płatności są wysyłane do nieautoryzowanej osoby.

### Mitygacja

Ustalać odbiorców z autorytatywnych rekordów konta i rezerwacji, autoryzować żądania powiadomień, używać zatwierdzonych template z kontrolowanymi zmiennymi, minimalizować ilość danych osobowych i sprawdzać, czy każde powiadomienie należy do właściwego odbiorcy oraz stanu workflow.

### Severity

High

### Status

Open

### STRIDE

Information Disclosure

### Uzasadnienie STRIDE

Wrażliwe dane rezerwacji, biletu lub płatności są ujawniane nieautoryzowanemu odbiorcy.

### Threat ID

T-015

### Tytuł

Wrażliwe dane rezerwacji lub płatności wysłane do złego odbiorcy

### Weryfikacja

Zmienić odbiorcę, booking ID i zmienne template w żądaniach wyzwalających powiadomienie. Użyć dwóch kont klientów i sprawdzić, czy każda wiadomość zawiera wyłącznie minimalne dane dozwolone dla właściwego klienta.

---

## T-016 - Flooding powiadomień przez nieograniczone operacje wysyłki

### Dotknięty element

Przepływy Booking Service i Payment Service → Notification Service

### Opis

Atakujący albo błędna pętla retry wielokrotnie wyzwala operacje powiadomień, zalewając odbiorców, zużywając limit providera i utrudniając dostarczenie prawidłowych wiadomości.

### Mitygacja

Udostępniać operacje powiadomień wyłącznie autoryzowanym serwisom wewnętrznym. Stosować deduplikację per event, rate limits, limity kolejki i retry backoff. Używać identyfikatorów idempotencji dla powiadomień biznesowych oraz monitorować nietypowy wolumen wysyłki i błędy providera.

### Severity

Medium

### Status

Open

### STRIDE

Denial of Service

### Uzasadnienie STRIDE

Powtarzane żądania powiadomień mogą wyczerpać capacity serwisu lub providera i zakłócić dostarczanie prawidłowych wiadomości.

### Threat ID

T-016

### Tytuł

Flooding powiadomień przez nieograniczone operacje wysyłki

### Weryfikacja

Ponowić i równolegle wyzwolić to samo zdarzenie powiadomienia wiele razy. System musi zdeduplikować zdarzenie biznesowe, egzekwować limity i zachować capacity dla prawidłowych wiadomości.

---

## T-017 - Nadmierne dane wrażliwe zwracane w odpowiedziach API

### Dotknięty element

Przepływy odpowiedzi Backend Services → API Gateway → Web Frontend

### Opis

Odpowiedzi API zawierają pola, których bieżący użytkownik lub frontend nie potrzebuje, takie jak wewnętrzne identyfikatory, prywatne dane uczestników, szczegóły ról, metadane płatności albo ukryte informacje o wydarzeniu.

### Mitygacja

Autoryzować przed serializacją, używać jawnych response DTO lub allowlist pól, zwracać minimalne dane wymagane dla bieżącej roli i akcji oraz nie traktować filtrowania po stronie frontendu jako kontroli bezpieczeństwa.

### Severity

High

### Status

Open

### STRIDE

Information Disclosure

### Uzasadnienie STRIDE

Backend zwraca wrażliwe pola klientowi, który nie ma prawa lub potrzeby ich otrzymać.

### Threat ID

T-017

### Tytuł

Nadmierne dane wrażliwe zwracane w odpowiedziach API

### Weryfikacja

Przechwycić odpowiedzi jako klient, organizer, administrator i niezalogowany użytkownik. Porównać pola i przetestować bezpośrednie wywołania API. Każda rola musi otrzymać wyłącznie jawnie dozwolone i potrzebne dane.

---

## T-018 - Eskalacja uprawnień przez niespójną autoryzację między endpointami

### Dotknięty element

API Gateway i wszystkie granice autoryzacji serwisów docelowych

### Opis

Wrażliwa akcja jest chroniona na jednej trasie, ale pozostaje dostępna przez alternatywny endpoint, inną wersję API, operację batch lub bezpośrednią ścieżkę do serwisu ze słabszą autoryzacją.

### Mitygacja

Zdefiniować centralną politykę autoryzacji i egzekwować ją niezależnie w każdym serwisie docelowym. Domyślnie odmawiać dostępu, utrzymywać macierz endpoint-to-permission, uwzględniać trasy alternatywne i legacy oraz dodać automatyczne negatywne testy autoryzacji dla każdej wrażliwej operacji.

### Severity

High

### Status

Open

### STRIDE

Elevation of Privilege

### Uzasadnienie STRIDE

Użytkownik dociera do uprzywilejowanej akcji przez słabiej chronioną ścieżkę i uzyskuje możliwości poza swoją rolą.

### Threat ID

T-018

### Tytuł

Eskalacja uprawnień przez niespójną autoryzację między endpointami

### Weryfikacja

Dla każdej wrażliwej operacji przetestować standardowe, alternatywne, legacy, batch i bezpośrednie trasy API, używając nieautoryzowanych ról oraz zmodyfikowanych resource ID. Każda ścieżka musi zwrócić 403 i nie zmienić stanu.
