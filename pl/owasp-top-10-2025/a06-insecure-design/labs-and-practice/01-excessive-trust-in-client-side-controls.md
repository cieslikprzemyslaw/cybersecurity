# Lab: Excessive Trust in Client-Side Controls

## Źródło

[PortSwigger lab](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)

## Zamierzona reguła biznesowa

Klient może kupić produkt wyłącznie po cenie ustalonej przez sklep i tylko wtedy, gdy ma wystarczający kredyt.

## Początkowa obserwacja

Request dodający produkt do koszyka zawierał:

```text
productId
quantity
price
```

`quantity` jest prawidłowo wybierane przez klienta, chociaż backend nadal musi walidować dozwolony zakres.

Najważniejsze pytanie: czy `price` jest wyłącznie danymi do wyświetlenia, czy autorytatywną wartością zaufaną przez serwer?

## Hipoteza

Backend może używać `price` przesłanego przez klienta zamiast pobierać cenę produktu z zaufanych danych serwerowych.

## Kontrolowany test

Zmieniona została wyłącznie cena:

```http
POST /cart

productId=2&redir=PRODUCT&quantity=1&price=0001
```

Serwer zwrócił redirect, a koszyk wyświetlił produkt za `$0.01`.

Było to więcej niż lokalna zmiana DOM, ponieważ zmodyfikowana wartość została zapisana w stanie koszyka po stronie serwera.

## Potwierdzenie wpływu

Drugi request dodał dziesięć sztuk produktu, którego normalna cena jednostkowa wynosiła `$1337.00`:

```http
POST /cart

productId=1&redir=PRODUCT&quantity=10&price=0001
```

Suma koszyka wyniosła `$0.10`, a checkout zakończył się powodzeniem.

## Fakty

- Request zawierał cenę kontrolowaną przez klienta.
- Serwer zaakceptował `price=0001`.
- Koszyk użył zmodyfikowanej ceny.
- Finalny checkout zaakceptował zmanipulowaną sumę.

## Niebezpieczne założenie

Design zakładał, że przeglądarka prześle prawidłową cenę sklepu.

## Przyczyna źródłowa

Wartość kontrolowana przez niezaufanego klienta została użyta jako autorytatywna wartość finansowa.

Problemem nie było wyłącznie to, że Burp pozwala edytować request. Serwer przekazał krytyczną decyzję biznesową niezaufanemu klientowi.

## Dlaczego to Insecure Design

Nawet technicznie poprawny kod byłby niebezpieczny, jeżeli zamierzony design mówi:

```text
odbierz product ID, quantity i price
  -> zaufaj trzem wartościom
  -> utwórz sumę koszyka
```

Wymagana kontrola musi istnieć w designie:

```text
odbierz product ID i quantity
  -> pobierz cenę z zaufanych danych serwerowych
  -> zwaliduj quantity
  -> oblicz i ponownie oblicz sumę po stronie serwera
```

## Wpływ

Atakujący może kupować produkty za zmanipulowaną kwotę, powodując bezpośrednią stratę finansową i niespójne rekordy zamówień.

## Wymaganie bezpieczeństwa

> Serwer musi obliczać ceny produktów i sumy zamówień z zaufanych danych produktowych po stronie serwera. Cena przesłana przez klienta musi zostać zignorowana albo odrzucona.

## Remediacja

- Usunąć autorytatywną cenę z kontraktu klienta.
- Pobierać cenę i promocje po serwerowo wybranym `productId`.
- Walidować quantity i dostępność produktu po stronie serwera.
- Ponownie obliczać kompletne zamówienie podczas checkout.
- Używać jednego autorytatywnego pricing service lub zestawu reguł dla cart i checkout.
- Logować odrzucone próby manipulacji jako detection, nie jako główny fix.

## Testy regresji

- Przesłać poprawny produkt i quantity z `price=0001`; serwer musi użyć prawidłowej ceny albo odrzucić pole.
- Usunąć pole price; cena nadal musi zostać poprawnie obliczona.
- Zmienić cenę po dodaniu produktu, ale przed checkout; checkout musi ponownie wyliczyć sumę.
- Przetestować quantity zero, ujemne i bardzo duże.
- Potwierdzić, że żaden alternatywny endpoint cart lub checkout nie akceptuje kwoty definiowanej przez klienta.

## Proces nauki

Początkowo skupiłem się na `quantity`, ponieważ było wyraźnie kontrolowane przez użytkownika.

Silniejszym kandydatem było `price`, ponieważ klient nie powinien podejmować finalnej decyzji cenowej. Decydującym dowodem nie był sam redirect, lecz utrwalona wartość koszyka i poprawny checkout.
