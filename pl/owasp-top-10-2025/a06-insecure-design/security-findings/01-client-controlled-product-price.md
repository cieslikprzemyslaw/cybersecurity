# Cena Produktu Kontrolowana przez Klienta Zaakceptowana przez Checkout

## Podsumowanie

Endpoint koszyka akceptował cenę produktu przesłaną przez klienta i używał jej do obliczenia sumy zamówienia.

Klient mógł zmienić pole `price` w żądaniu dodającym produkt do koszyka i zakończyć checkout za zmanipulowaną kwotę.

## Sugerowane severity

High

Dokładne severity zależy od wartości produktów, settlement płatności, fraud monitoring i tego, czy zamówienia są automatycznie realizowane. Autoryzowany lab potwierdził bezpośredni wpływ finansowy, ponieważ produkt o wysokiej wartości został zakupiony za minimalną kwotę.

## Dotknięty komponent

```text
POST /cart
Parameter: price
Checkout workflow
```

## Podatne zachowanie

Klient przesyłał:

```text
productId
quantity
price
```

Serwer traktował przesłane `price` jako autorytatywne zamiast pobierać cenę produktu z zaufanych danych produktowych po stronie serwera.

Konceptualny vulnerable flow:

```text
client wybiera produkt
  -> client przesyła product ID, quantity i price
  -> server ufa przesłanej cenie
  -> cart zapisuje zmanipulowaną kwotę
  -> checkout akceptuje zmanipulowaną sumę
```

## Dowody

Produkt został dodany przy użyciu:

```http
productId=2&redir=PRODUCT&quantity=1&price=0001
```

Koszyk wyświetlił produkt za `$0.01`.

Drugi test użył:

```http
productId=1&redir=PRODUCT&quantity=10&price=0001
```

Normalna cena jednostkowa pokazywana przez aplikację wynosiła `$1337.00`, ale suma koszyka wyniosła `$0.10`.

Checkout zakończył się powodzeniem.

Potwierdziło to, że zmodyfikowana wartość wpłynęła na przetwarzanie zamówienia po stronie serwera, a nie wyłącznie na lokalny UI.

## Przyczyna źródłowa

Wartość kontrolowana przez niezaufanego klienta została użyta jako autorytatywna wartość finansowa.

Brakującym design requirement było obliczanie ceny i sumy z zaufanych danych po stronie serwera.

## Dlaczego to Insecure Design

Niebezpieczny rezultat może wystąpić nawet wtedy, gdy kod idealnie implementuje kontrakt żądania.

Jeżeli design celowo akceptuje i ufa autorytatywnej cenie klienta, lokalny patch walidacyjny nie naprawia decyzji o zaufaniu. Kontrakt i flow obliczeń muszą się zmienić.

## Wpływ

Atakujący może potencjalnie:

- kupować produkty poniżej zamierzonej ceny,
- tworzyć niespójne rekordy zamówienia i płatności,
- powodować bezpośrednią stratę przychodu,
- nadużywać automatycznego fulfilment,
- połączyć problem ze słabościami quantity albo promotions.

Lab potwierdził udany zakup za zmanipulowaną kwotę.

## Wymaganie bezpieczeństwa

> Serwer musi obliczać ceny produktów i kompletne sumy zamówień z zaufanych danych produktowych i promocyjnych po stronie serwera. Wartości price przesłane przez klienta nie mogą wpływać na autorytatywną kwotę.

## Remediacja

1. Usunąć autorytatywne pole `price` z żądania klienta.
2. Pobierać aktualną cenę produktu po server-selected `productId`.
3. Walidować ilość, dostępność i uprawnienie do promocji po stronie serwera.
4. Ponownie obliczać pełną sumę podczas checkout.
5. Używać tych samych autorytatywnych reguł pricing w cart, checkout i fulfilment.
6. Powiązać wynikową kwotę z zamówieniem i operacją płatności.
7. Bezpiecznie odrzucać niespójny lub stale pricing state.
8. Logować próby manipulacji dla detection, nie traktując logowania jako fixu.

## Testy regresji

- Przesłać `price=0001`; zapisana i pobrana kwota musi pozostać prawidłową ceną serwerową.
- Pominąć pole `price`; zamówienie nadal musi zostać prawidłowo wycenione.
- Zmienić cenę po utworzeniu koszyka; checkout musi ponownie obliczyć sumę.
- Zmienić quantity na zero, wartość ujemną i bardzo dużą.
- Przetestować wszystkie cart, checkout, API-version i batch routes.
- Potwierdzić, że odrzucone albo zignorowane wartości nie zmieniają stanu zamówienia.
- Potwierdzić, że payment amount odpowiada finalnej kwocie obliczonej przez serwer.

## Podsumowanie autoryzowanej reprodukcji

1. Dodać produkt normalnie i przechwycić request.
2. Zidentyfikować `productId`, `quantity` i `price`.
3. Zmienić wyłącznie `price` na minimalną dodatnią wartość.
4. Potwierdzić, że cart utrwala zmienioną kwotę.
5. Zakończyć checkout.
6. Zapisać normalną cenę, zmanipulowaną sumę i wynik udanego zamówienia.
7. Oddzielić potwierdzone dowody od założeń dotyczących backend implementation.

## Referencje

- [OWASP A06:2025 - Insecure Design](https://owasp.org/Top10/2025/A06_2025-Insecure_Design/)
- [PortSwigger Business Logic Vulnerabilities](https://portswigger.net/web-security/logic-flaws)
- [PortSwigger lab: Excessive trust in client-side controls](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)
