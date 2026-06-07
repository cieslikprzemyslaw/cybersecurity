# SSTI Overview

## Czym jest server-side template engine?

Server-side template engine laczy szablon kontrolowany przez developera z danymi, aby wygenerowac output, na przyklad HTML, email, dokument albo tekst konfiguracyjny.

Przyklad koncepcyjny:

```text
statyczny szablon: Hello, {{ name }}
dane:              name = "Przemyslaw"
wynik:             Hello, Przemyslaw
```

Aplikacje uzywaja szablonow, zeby nie budowac recznie kazdej strony i oddzielic warstwe prezentacji od logiki aplikacji.

## Czym jest SSTI?

SSTI wystepuje wtedy, gdy niezaufany input zostaje dolaczony do zrodla szablonu, a silnik interpretuje go jako skladnie albo wyrazenie.

```text
bezpiecznie:
statyczny szablon + input uzytkownika jako dane

niebezpiecznie:
template source zbudowany z inputu uzytkownika + compile/render
```

## Poziomy dowodow

### 1. Zwykle odbicie

Input pojawia sie w odpowiedzi dokladnie tak, jak zostal wyslany.

```text
input:  SSTI_TEST_123
output: SSTI_TEST_123
```

To potwierdza kontrole inputu i lokalizacje outputu. Nie potwierdza SSTI.

### 2. Renderowanie HTML

Przegladarka interpretuje wygenerowany markup. To moze byc istotne dla XSS, ale samo w sobie nie dowodzi, ze server-side template engine ocenil wyrazenie.

### 3. Ewaluacja wyrazenia szablonu

Wyrazenie szablonu zostaje zastapione obliczonym wynikiem.

```text
input:  engine-specific expression for 7 * 7
output: 49
```

To jest dowod, ze server-side template engine ocenil wyrazenie.

### 4. Glebszy impact

Dalsze testy moga pokazac dostep do:

- zmiennych template context,
- obiektow aplikacji,
- zmiennych srodowiskowych,
- plikow,
- API runtime jezyka,
- API uruchamiania procesow.

Sama ewaluacja wyrazenia nie dowodzi automatycznie remote code execution. Kazda glebsza mozliwosc wymaga osobnego dowodu.

## SSTI a XSS

| Pytanie | SSTI | XSS |
|---|---|---|
| Glowny interpreter | Server-side template engine | Przegladarka / JavaScript engine |
| Moment przetwarzania | Przed wygenerowaniem odpowiedzi | Po dotarciu odpowiedzi do przegladarki |
| Typowy dowod | Wyrazenie ocenione do wyniku | Wykonanie skryptu albo zachowania przegladarki |
| Glowny cel | Aplikacja serwerowa i jej uprawnienia | Sesja uzytkownika i browser context |

HTML escaping moze pomagac przeciw XSS w wygenerowanym outputcie, ale nie naprawia sytuacji, w ktorej input uzytkownika staje sie wykonywalnym zrodlem szablonu.

## SSTI a OS Command Injection

| Pytanie | SSTI | OS Command Injection |
|---|---|---|
| Poczatkowy root cause | Input zmienia ewaluacje szablonu | Input zmienia komende shella albo wywolanie procesu |
| Poczatkowy interpreter | Template engine | Shell, command interpreter albo niebezpieczne process API |
| Mozliwa eskalacja | Dostep do runtime moze prowadzic do process execution | Wykonanie komendy jest juz podatnym zachowaniem |

SSTI moze prowadzic do OS command execution, ale command execution jest sciezka impactu. Nie zmienia pierwotnego root cause.

## Potencjalny impact

W zaleznosci od silnika, runtime, context, konfiguracji i uprawnien aplikacji impact moze obejmowac:

- zmiane tresci generowanej po stronie serwera,
- information disclosure,
- dostep do konfiguracji lub sekretow aplikacji,
- odczyt, zapis albo usuwanie plikow,
- server-side code execution,
- uruchamianie procesow systemu operacyjnego,
- kompromitacje danych dostepnych dla procesu aplikacji.

Impact jest ograniczony przez uprawnienia i srodowisko procesu aplikacji. SSTI nie oznacza automatycznie dostepu root ani administratora.

## Root cause

Root cause to blad granicy dane/kod:

> Dane kontrolowane przez uzytkownika zostaly potraktowane jako template source, zamiast zostac przekazane jako zwykla zmienna do statycznego szablonu.

To jest precyzyjniejsze niz samo stwierdzenie, ze "input nie byl sanityzowany".
