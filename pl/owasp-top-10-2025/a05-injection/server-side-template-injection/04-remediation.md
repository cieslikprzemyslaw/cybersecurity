# SSTI Remediation

## Glowna poprawka

Nigdy nie pozwalaj, aby tresc kontrolowana przez uzytkownika stala sie template source.

```text
bezpiecznie:
statyczny szablon kontrolowany przez developera
  + wartosci kontrolowane przez uzytkownika przekazane jako dane

niebezpiecznie:
statyczny tekst + input uzytkownika
  -> dynamic template compilation
```

## Zasady secure design

### 1. Trzymaj szablony jako statyczne i kontrolowane przez developerow

Przechowuj szablony w reviewowanych plikach albo kontrolowanych zasobach. Nie doklejaj query parameters, wartosci formularza, pol CMS ani wartosci profilu do template source.

### 2. Przekazuj input tylko jako variables albo locals

Engine powinien dostac zaufany szablon i osobny obiekt danych.

Przyklad koncepcyjny:

```text
render(template='out-of-stock', data={ message: safeMessage })
```

Wartosc zmiennej nie moze byc parsowana ponownie jako drugi szablon.

### 3. Preferuj identyfikatory zamiast dowolnego display text

Zamiast przyjmowac pelny komunikat od klienta:

```text
?message=Unfortunately this product is out of stock
```

przyjmij ograniczony identyfikator:

```text
?status=out_of_stock
```

Serwer wybiera finalny komunikat ze stalej mapy albo katalogu tlumaczen.

### 4. Unikaj dynamic compilation APIs

Sprawdz wywolania, ktore kompilują albo renderuja szablony ze stringow, szczegolnie gdy string zawiera dane z requestu.

Ryzykowne wzorce koncepcyjnie:

- `from_string(userControlledValue)`
- `compile(prefix + userControlledValue)`
- `render(userControlledTemplateSource)`

Dokladne API zalezy od frameworka i jezyka.

### 5. Ogranicz template context

Wystawiaj tylko minimalne dane i helper functions potrzebne szablonowi.

Nie wystawiaj:

- process objects,
- environment objects,
- filesystem helpers,
- database clients,
- powerful service containers,
- sekretow ani obiektow konfiguracji.

### 6. Sandbox tylko jako defence in depth

Sandbox moze ograniczac atrybuty, metody, funkcje, dostep do plikow albo cechy jezyka, gdy uzytkownicy rzeczywiscie musza tworzyc szablony.

Nie jest glowna poprawka dla aplikacji, ktora nigdy nie potrzebowala user-authored templates.

Ryzyka sandboxa:

- niepelne policies,
- niebezpieczne allowed helpers,
- bypassy zalezne od wersji,
- zbyt duzo obiektow w template context,
- denial-of-service przez kosztowne wyrazenia.

### 7. Output escaping jest dla XSS, nie jako fix SSTI

HTML escaping chroni kontekst wygenerowanego outputu. Nie zapobiega ewaluacji server-side expression przed escapowaniem outputu.

### 8. Stosuj least privilege

Uruchamiaj aplikacje tylko z tymi uprawnieniami filesystemu, sieci i systemu operacyjnego, ktorych potrzebuje.

To zmniejsza impact, jesli blad szablonu dotrze do file albo process APIs.

### 9. Unikaj verbose errors na produkcji

Zwracaj uzytkownikom generyczne bledy, a szczegoly template exceptions trzymaj w chronionych logach serwerowych.

### 10. Utrzymuj i reviewuj template engines

Aktualizuj engine i framework. Reviewuj security configuration, dozwolone plugins, functions, modifiers, globals i custom extensions.

## Engine-specific defence-in-depth

### Jinja2

- Najpierw statyczne templates i data variables.
- Gdy user-authored templates sa celowa funkcja, rozwaz `SandboxedEnvironment`.
- Ogranicz globals, filters, tests i callable objects.
- Pamietaj, ze autoescape pomaga glownie z XSS output handling.

### Pug

- Nie kompiluj user-controlled strings jako Pug source.
- Przekazuj wartosci jako locals do statycznego template.
- Preferuj escaped output dla niezaufanych wartosci.
- Nie wystawiaj poteznych Node.js runtime objects ani process helpers.

### Smarty

- Wlacz i skonfiguruj odpowiednia security policy, gdy untrusted templates sa celowa funkcja.
- Allowlistuj bezpieczne modifiers, functions, plugins i resources.
- Nie traktuj wylaczenia jednego tagu jako pelnej granicy bezpieczenstwa.

### ERB

- Nie buduj ERB template przez konkatenacje request values.
- Renderuj statyczne `.erb` templates ze zmiennymi przekazywanymi osobno.
- Nie wystawiaj dowolnych obiektow Ruby ani helper methods niezaufanym autorom szablonow.

## Dlaczego "sanitize input" nie wystarcza

Blacklist delimiterow albo keywordow jest krucha, bo:

- engines maja wiele form skladni,
- encoding i transformacje moga zmienic input,
- kontekst wplywa na parsowanie,
- legalny tekst moze zostac uszkodzony,
- przyszle funkcje engine moga dac nowe bypassy.

Walidacja jest przydatna dla business rules. Granica bezpieczenstwa musi polegac na oddzieleniu template source od danych.

## Pytania do code review

- Czy jakakolwiek wartosc z requestu tworzy template source?
- Czy aplikacja kompiluje templates ze stringow?
- Czy zapisane CMS albo profile content moze stac sie template syntax?
- Czy jakakolwiek renderowana wartosc jest ewaluowana wiecej niz raz?
- Jakie obiekty i helpery sa dostepne w szablonie?
- Czy uzytkownicy celowo moga tworzyc templates?
- Jesli tak, jaki sandbox, allowlist, quota i isolation controls istnieja?
- Jakie uprawnienia filesystemu i sieci ma proces?
