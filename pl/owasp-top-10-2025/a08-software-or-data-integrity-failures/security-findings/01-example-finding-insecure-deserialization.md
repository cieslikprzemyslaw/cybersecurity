# Serializowany stan sesji kontrolowany przez klienta umożliwia privilege escalation

## Severity

**High**

## Kategoria

- OWASP Top 10 2025: A08 Software or Data Integrity Failures
- Powiązany wpływ: Broken Access Control / privilege escalation
- Powiązane wzorce weakness: deserialization of untrusted data oraz reliance on cookies without integrity checking

## Podsumowanie

Aplikacja przechowuje serializowany obiekt `User` w session cookie kontrolowanym przez klienta. Obiekt zawiera boolean `admin`, któremu backend ufa podczas autoryzacji dostępu do funkcji administracyjnych.

Cookie jest zakodowane przez URL i Base64, ale testowany flow nie posiada skutecznej ochrony integralności. Zwykły użytkownik może odkodować obiekt, zmienić `admin` z `false` na `true`, zakodować ponownie i uzyskać uprawnienia administratora.

## Affected data flow

```text
session cookie kontrolowane przez przeglądarkę
    -> URL decoding
    -> Base64 decoding
    -> deserializacja obiektu PHP
    -> zaufana właściwość admin
    -> decyzja autoryzacji
```

## Dowody

Oryginalny obiekt:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

Zmieniony obiekt:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

Po przesłaniu zmienionego cookie:

1. response pokazał link `/admin`,
2. `GET /admin` zwrócił panel administratora,
3. `GET /admin/delete?username=carlos` wykonał akcję i przekierował do `/admin`.

## Wykazany wpływ

Zwykły uwierzytelniony użytkownik mógł:

- podnieść uprawnienia do administratora,
- uzyskać dostęp do funkcji administracyjnych,
- usunąć innego użytkownika.

Remote code execution nie było testowane ani wykazane.

## Root cause

Aplikacja traktuje właściwość istotną z punktu widzenia bezpieczeństwa z deserializowanego obiektu kontrolowanego przez klienta jako autorytatywną. Nie weryfikuje wystarczająco integralności obiektu i nie wyprowadza niezależnie uprawnień administracyjnych z zaufanego stanu po stronie serwera.

## Security requirement

> Dane sesji kontrolowane przez klienta nie mogą ustalać ról ani uprawnień użytkownika. Autoryzacja administracyjna musi pochodzić z zaufanego stanu po stronie serwera i być egzekwowana na każdym chronionym endpoincie.

Dodatkowy requirement:

> Jeśli jakikolwiek zaufany stan jest przechowywany po stronie klienta, jego integralność i autentyczność muszą zostać zweryfikowane przed parsowaniem, deserializacją lub użyciem. Niepoprawny, brakujący, wygasły lub odtworzony stan musi fail closed.

## Zalecana remediation

### Primary remediation

- Zastąpić serializowany obiekt sesji losowym opaque session identifier.
- Przechowywać identity, role i permissions w chronionym server-side session store albo bazie.
- Egzekwować autoryzację na każdym endpointcie administracyjnym.
- Unikać natywnej deserializacji obiektów od niezaufanych klientów.
- Używać prostego formatu z jawnym schematem dla niezbędnego inputu klienta.

### Defence in depth

- Jeśli stan klienta jest wymagany, chronić go silnym MAC albo podpisem cyfrowym.
- Weryfikować integralność przed deserializacją lub zaufanym przetwarzaniem.
- Dodać expiry i właściwe context binding.
- Rotować i chronić signing keys.
- Logować błędy integralności i powtarzające się próby modyfikacji.
- Odrzucać nieoczekiwane typy i pola.
- Stosować least privilege dla komponentu przetwarzającego dane.

## Testy regresyjne

1. Zmienić `admin=false` na `admin=true`; użytkownik nadal nie może być adminem.
2. Wywołać `/admin` ze zmienionym stanem klienta; dostęp ma być odrzucony.
3. Wywołać admin endpoint bezpośrednio; autoryzacja po stronie serwera ma odrzucić akcję.
4. Usunąć lub uszkodzić wartość integralności; oczekiwane bezpieczne odrzucenie.
5. Wysłać malformed albo unexpected serialized object; kontrolowane odrzucenie przed niebezpieczną rekonstrukcją.
6. Odtworzyć expired signed state; oczekiwane odrzucenie.
7. Potwierdzić logowanie błędów integralności bez ujawniania wrażliwych danych.

## Developer takeaway

Base64 i serializacja są reprezentacją danych, a nie kontrolą zaufania. Security-sensitive authority musi pochodzić z zaufanego stanu po stronie serwera.
