# Lab: Flawed Enforcement of Business Rules

## Źródło

[PortSwigger lab](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-flawed-enforcement-of-business-rules)

## Zamierzona reguła biznesowa

Kody promocyjne muszą przestrzegać zdefiniowanej polityki użycia i łączenia.

Kod przeznaczony dla nowego klienta albo do jednorazowego użycia nie może ponownie stać się dostępny tylko dlatego, że pomiędzy próbami zastosowano inny prawidłowy kod.

## Zaobserwowane promocje

Aplikacja ujawniała dwa prawidłowe kody:

```text
NEWCUST5
SIGNUP30
```

`NEWCUST5` obniżał zamówienie o `$5.00`.

`SIGNUP30` obniżał cenę o 30%.

## Pierwsze założenie

Backend może prawidłowo odrzucać kupon, jeśli ten sam kod był już użyty.

Bezpośredni duplikat wyglądał na ograniczony, ale nie dowodziło to, że walidowana jest pełna historia kuponów.

## Kontrolowana sekwencja

Aplikacja zaakceptowała:

```text
NEWCUST5
SIGNUP30
NEWCUST5
SIGNUP30
NEWCUST5
SIGNUP30
NEWCUST5
SIGNUP30
```

Suma zamówienia produktu o cenie `$1337.00` spadła do `$0.00`.

Checkout zakończył się powodzeniem.

## Fakty

- Oba kody były prawidłowe.
- Aplikacja akceptowała oba typy kuponów w tym samym koszyku.
- Naprzemienne używanie kodów pozwoliło stosować ten sam kod wiele razy.
- Kompletne zamówienie zostało zaakceptowane za `$0.00`.

## Wniosek

Zachowanie mocno sugeruje, że wykrywanie duplikatu uwzględniało wyłącznie ostatnio zastosowany kupon albo w inny sposób nie walidowało pełnego stanu promocji.

Dokładna implementacja nie może zostać potwierdzona bez kodu źródłowego.

## Niebezpieczne założenie

Workflow zakładał, że zablokowanie dwóch identycznych kodów bezpośrednio obok siebie wystarczy do egzekwowania polityki promocji.

## Przyczyna źródłowa

Aplikacja nie egzekwowała reguł użycia kuponów w pełnym autorytatywnym stanie koszyka, zamówienia lub historii klienta wymaganej przez promocję.

Problemem nie był format kuponu. Każdy input był prawidłowy.

## Dlaczego input validation nie naprawi problemu

Input validation może potwierdzić, że:

- kod istnieje,
- ma prawidłową składnię,
- jest aktywny,
- nie wygasł.

Nie naprawi modelu stanu, który pozwala na:

```text
coupon A -> coupon B -> coupon A
```

Backend musi walidować historię, zasady łączenia i zakres użycia.

## Wymaganie bezpieczeństwa

> Serwer musi egzekwować politykę użycia i łączenia każdego kuponu w pełnym autorytatywnym stanie koszyka, zamówienia i historii klienta zdefiniowanej przez promocję, łącznie z finalną walidacją podczas checkout.

## Remediacja

- Zdefiniować, czy każdy kupon ma limit na konto, koszyk, zamówienie lub okres promocji.
- Przechowywać wszystkie zastosowane promocje jako autorytatywny stan serwerowy.
- Odrzucać kod wykorzystany już w jego zdefiniowanym zakresie.
- Egzekwować dozwolone i zabronione kombinacje kuponów.
- Obliczać rabaty z serwerowych reguł promocji.
- Ponownie walidować pełny zestaw promocji podczas checkout.
- Zapewnić bezpieczeństwo przy równoległym stosowaniu kuponów i checkout.

## Testy regresji

- Zastosować `NEWCUST5`, potem `SIGNUP30`, a następnie ponownie `NEWCUST5`; drugie użycie musi zostać odrzucone.
- Sprawdzić bezpośrednie duplikaty i naprzemienne duplikaty.
- Powtórzyć test przez każdą trasę cart i checkout.
- Ponowić finalny request kuponu.
- Wysłać checkout ze zmanipulowaną lub nieaktualną listą promocji.
- Przetestować równoległe stosowanie kuponów.
- Potwierdzić, że odrzucone żądania nie zmieniają sumy koszyka.

## Proces nauki

Najpierw zauważyłem, że drugi legalny kupon może być istotny.

Najważniejszym ulepszeniem było przejście od zgadywania kodów do testowania konkretnej reguły biznesowej. Podatność pojawiła się przez sekwencję i historię stanu, a nie przez malformed payload.
