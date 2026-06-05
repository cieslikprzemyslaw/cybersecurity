# Threat Modeling Mini-Project Debrief

## Co zamodelowalem

Utworzylem prosty threat model aplikacji webowej z nastepujaca architektura:

- User
- Browser
- React Frontend
- Backend API
- Database

Dodalem dwie granice zaufania:

- granica po stronie klienta
- granica po stronie serwera

Glowne przeplywy danych:

- interakcja uzytkownika z przegladarka
- ladowanie i wykonywanie aplikacji React w przegladarce
- renderowanie i aktualizacja DOM przez React
- requesty HTTPS API z frontendu do backendu
- odpowiedzi API z backendu do frontendu
- zapytania backendu do bazy danych
- wyniki zapytan zwracane z bazy danych

## Dlaczego uzylem STRIDE

Uzylem STRIDE, zeby uporzadkowac potencjalne zagrozenia wedlug kategorii:

- Spoofing
- Tampering
- Repudiation
- Information Disclosure
- Denial of Service
- Elevation of Privilege

Dzieki temu moglem przejrzec cala architekture, zamiast skupic sie tylko na injection albo problemach z uwierzytelnianiem.

## Zidentyfikowane zagrozenia

1. Modified API Request
2. Broken Object-Level Authorization
3. Session or Token Theft
4. Excessive Data Returned by API
5. Missing Audit Logging
6. Sensitive Information Exposed in Browser Console
7. Injection in Database Query
8. Sensitive Data Returned from Database

## Kluczowe zalozenia bezpieczenstwa

- Uzytkownik i przegladarka sa niezaufane.
- Requesty po stronie klienta moga zostac zmodyfikowane przed dotarciem do backendu.
- Backend musi samodzielnie ustalic uwierzytelnionego uzytkownika i egzekwowac autoryzacje.
- Baza danych powinna byc dostepna przez backend.
- Odpowiedzi API powinny stosowac minimalizacje danych i zasade najmniejszych uprawnien.
- HTTPS chroni dane w tranzycie, ale nie zastepuje kontroli bezpieczenstwa na poziomie aplikacji.

## Czego sie nauczylem

Najwazniejsza lekcja: threat modeling nie potwierdza istnienia podatnosci. Pomaga wskazac miejsca, w ktorych slabosc moze sie pojawic, oraz co trzeba pozniej przejrzec albo przetestowac.

Dodatkowo:

- poprawna sesja nie oznacza automatycznie, ze uzytkownik ma dostep do kazdego obiektu
- zapytania parametryzowane chronia przed injection, ale nie chronia przed nadmiernym ujawnianiem danych
- audit logging wspiera rozliczalnosc, ale logi nie moga zawierac sekretow ani niepotrzebnych danych osobowych
- odpowiedzi API powinny stosowac minimalizacje danych i zasade najmniejszych uprawnien
- logowanie do konsoli przegladarki moze ujawniac wrazliwe informacje
- granice zaufania pomagaja znalezc miejsca, gdzie dane przechodza miedzy komponentami o roznym poziomie zaufania
- jakosc threat modelu zalezy mocno od tego, jak dokladnie opisano architekture i przeplywy danych

## Pomysly na walidacje

Zagrozenia mozna walidowac przez:

- manualny code review
- SAST
- DAST
- testowanie API
- przeglad konfiguracji sesji i cookies
- testy autoryzacji
- przeglad schematow odpowiedzi
- przeglad zapytan do bazy danych
- przeglad logow audytowych

## Ograniczenia

To byl hipotetyczny model edukacyjny, a nie model utworzony na podstawie realnej implementacji aplikacji.

Zidentyfikowane zagrozenia nie zostaly przetestowane na kodzie zrodlowym, infrastrukturze, konfiguracji ani dzialajacym systemie. Wszystkie pozostaja otwarte i powinny byc traktowane jako potencjalne ryzyka, nie potwierdzone podatnosci.

## Finalna refleksja

To cwiczenie pomoglo mi zrozumiec roznice miedzy:

- architektura
- granicami zaufania
- zagrozeniami
- podatnosciami
- walidacja
- mitygacjami

Pokazalo tez, ze threat modeling jest najbardziej przydatny przed developmentem albo podczas design review, bo pomaga wskazac wymagania bezpieczenstwa zanim slabosci trafia na produkcje.
