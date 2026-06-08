# A06: Insecure Design - Learning Notes

## Początkowy mental model

Na początku opisywałem Insecure Design zbyt szeroko jako aplikację, która nie została poprawnie zabezpieczona.

Bardziej użyteczna definicja:

> Design nie zawiera skutecznej kontroli, używa niebezpiecznego założenia albo pozwala na workflow, który nie powinien być możliwy.

Brak dowodu nie jest automatycznie potwierdzonym design flaw. Jest niezweryfikowanym założeniem, dopóki architektura, kod lub zachowanie nie dostarczą dowodu.

## Korekty, które miały znaczenie

### Design versus implementacja

Nauczyłem się, że kod może być technicznie poprawny i nadal implementować insecure design.

Developer może poprawnie odebrać pole `price`, zapisać je i obsłużyć checkout. Design pozostaje niebezpieczny, jeżeli klient może przesłać autorytatywną cenę.

### Assets versus components

Początkowo wymieniałem Auth Service, Booking Service i bazy danych jako zasoby.

Komponenty przetwarzają lub przechowują zasoby. Zasoby to wartościowe tożsamości, role, rezerwacje, bilety, widoczność wydarzeń, płatności i stan biznesowy.

### Fakty versus założenia

Stwierdzenie developera, że token resetu jest „chyba jednorazowy”, nie jest weryfikacją.

Dobre review zapisuje:

- co wiadomo,
- co jest założeniem,
- co może się wydarzyć, jeżeli założenie jest błędne,
- co system musi egzekwować,
- jak to przetestować.

## Perspektywa frontendowa

Conditional rendering jest realny i przydatny:

```tsx
{isAdmin && <AdminPanel />}
```

Komponent nie jest renderowany, gdy warunek jest false. Poprawia to UX i ogranicza przypadkowe akcje.

Nie jest to autoryzacja, ponieważ użytkownik może nadal:

- bezpośrednio wywołać API,
- zmodyfikować request,
- zmienić stan w przeglądarce,
- otrzymać wrażliwe dane już zwrócone przez API.

Moje doświadczenie frontendowe pomaga rozpoznawać browser/API trust boundary, client-controlled state i miejsca, gdzie zespół może pomylić prezentację z enforcementem.

## Lekcje business logic

Pierwszy lab używał oczywiście niebezpiecznej wartości kontrolowanej przez klienta.

Drugi był mniej oczywisty. Każdy kupon był prawidłowy i każdy pojedynczy request został zaakceptowany. Podatność pojawiła się dopiero przez sekwencję:

```text
NEWCUST5 -> SIGNUP30 -> NEWCUST5 -> SIGNUP30
```

Pokazało to, dlaczego payload guessing i input validation nie wystarczają przy business logic. Reviewer musi rozumieć stan, limity, kolejność i powtarzanie.

## Lekcje threat modelling

Najbardziej użyteczną częścią threat modelling nie było rysowanie każdego możliwego komponentu.

Użyteczna sekwencja:

```text
asset
  -> actor i trust boundary
  -> unsafe assumption
  -> abuse case
  -> missing control
  -> security requirement
  -> weryfikacja
```

STRIDE pomagał klasyfikować zagrożenia, ale wybór kategorii wymagał rozpoznania głównego efektu bezpieczeństwa:

- zmiana autorytatywnych danych: Tampering,
- podszycie się pod tożsamość lub providera: Spoofing,
- ujawnienie prywatnych danych: Information Disclosure,
- rozszerzenie uprawnień: Elevation of Privilege,
- wyczerpanie capacity serwisu: Denial of Service.

## Lekcja narzędziowa

Duży diagram w Threat Dragon stał się trudny do czytania. Draw.io lepiej sprawdził się dla pełnego system context i DFD.

Threat Dragon nadal może być przydatny dla mniejszych diagramów konkretnego workflow i wbudowanych pól threats, ale architekturę trzeba dzielić, gdy diagram robi się zbyt gęsty.

## Finalny takeaway

Nie muszę pracować jako senior security architect, aby uczestniczyć w użytecznym design review.

Na obecnym poziomie FE-to-AppSec powinienem umieć:

- zidentyfikować asset i business rule,
- traktować browser jako niezaufany,
- kwestionować assumptions,
- zapytać, gdzie działa authorization i state validation,
- zaproponować testowalne security requirement,
- zaprojektować negative lub abuse-case test,
- odróżnić prevention od monitoring.
