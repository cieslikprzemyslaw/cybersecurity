# A08 Overview

## Definicja

Software or Data Integrity Failures występują, gdy aplikacja traktuje software, kod, konfigurację, aktualizacje, artefakty, serializowane obiekty albo krytyczne dane jako zaufane bez odpowiedniej weryfikacji:

- skąd pochodzą,
- czy zostały zmienione,
- czy są autentyczne,
- czy nadal są ważne,
- czy można je odtworzyć ponownie,
- czy nadawca był uprawniony do ich utworzenia lub zmiany.

Błąd pojawia się na granicy zaufania:

```text
niezaufany lub zewnętrzny artefakt
    -> brakująca albo niewystarczająca weryfikacja
    -> zaufane przetwarzanie
```

## Typowe przykłady

- niepodpisane aktualizacje software albo firmware,
- deployment artefaktów bez weryfikacji podpisu lub pochodzenia,
- joby CI/CD pobierające artefakty z niezaufanych lokalizacji,
- zewnętrzny JavaScript ładowany z CDN bez odpowiednich kontroli,
- cookies kontrolowane przez klienta traktowane jako zaufany stan sesji lub autoryzacji,
- serializowane obiekty przyjmowane od użytkowników i deserializowane,
- krytyczna konfiguracja lub feature flags bez ochrony integralności,
- rola, cena, konto, ścieżka lub uprawnienie przesłane przez przeglądarkę i zaakceptowane jako źródło prawdy.

## Czym A08 nie jest

A08 nie oznacza, że każde edytowalne cookie, JSON albo wartość przeglądarkowa jest automatycznie podatna.

Podatność istnieje, gdy:

1. wartość przekracza granicę do zaufanego komponentu,
2. aplikacja traktuje ją jako autorytatywną,
3. brakuje weryfikacji integralności lub autentyczności,
4. zmieniona wartość wpływa na zachowanie związane z bezpieczeństwem.

Nie każdy zmodyfikowany request JSON jest insecure deserialization. Niebezpieczna deserializacja dotyczy rekonstrukcji obiektów lub struktur z serializowanych danych kontrolowanych przez użytkownika, a następnie zaufania tym obiektom lub zachowaniu uruchamianemu podczas rekonstrukcji.

## Wpływ

Wpływ zależy od rodzaju artefaktu i sposobu jego użycia:

- privilege escalation,
- nieautoryzowane akcje administracyjne,
- dowolny dostęp do plików,
- wykonanie złośliwego kodu,
- deployment zmienionego buildu,
- użycie niebezpiecznej konfiguracji,
- denial of service,
- replay wcześniej poprawnych danych.

Wpływ musi wynikać z dowodów. Sama niebezpieczna deserializacja nie udowadnia automatycznie remote code execution.

## Główne zabezpieczenia

- Unikać natywnej deserializacji obiektów dla niezaufanego inputu.
- Preferować proste formaty z jawnym schematem.
- Rekonstruować wyłącznie oczekiwane typy i pola.
- Przechowywać wrażliwy autorytet po stronie serwera.
- Weryfikować podpisy lub MAC przed użyciem zaufanego stanu klienta.
- Weryfikować aktualizacje i artefakty builda przed wykonaniem lub deploymentem.
- Korzystać z zatwierdzonych źródeł, repozytoriów i originów.
- Po błędzie weryfikacji bezpiecznie odrzucać.
- Egzekwować autoryzację na endpointach niezależnie od stanu frontendu.

## Wyjaśnienie rekrutacyjne

> A08 dotyczy przekroczenia granicy zaufania bez dowodu, że software lub dane są autentyczne i niezmienione. Jako frontend developer sprawdziłbym role, ceny, właściwości sesji, feature flags i skrypty zewnętrzne kontrolowane przez przeglądarkę oraz artefakty builda, którym backend albo proces deploymentu ufa bez wystarczającej weryfikacji integralności.
