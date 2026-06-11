# Integralność i autentyczność

## Integralność

Integralność oznacza, że software lub dane nie zostały zmienione bez autoryzacji.

Najważniejsze pytanie:

> Czy to dokładnie ta zawartość, której oczekiwał zaufany komponent?

## Autentyczność

Autentyczność oznacza, że software lub dane pochodzą z oczekiwanego źródła.

Najważniejsze pytanie:

> Czy zostały utworzone lub zatwierdzone przez oczekiwaną tożsamość albo klucz podpisujący?

## Poufność

Poufność zapobiega nieautoryzowanemu ujawnieniu.

Najważniejsze pytanie:

> Czy nieuprawniona osoba może odczytać dane?

Te właściwości są powiązane, ale nie są tym samym.

## Kodowanie nie jest zabezpieczeniem

Base64, URL encoding i serializacja zmieniają reprezentację. Nie zapewniają:

- integralności,
- autentyczności,
- poufności,
- autoryzacji.

Przykład:

```text
admin=false
    -> Base64
    -> odwracalna reprezentacja
```

Atakujący może ją odkodować, zmienić i zakodować ponownie.

## Checksum

Checksum albo hash może wykryć, czy zawartość różni się od oczekiwanej:

```text
SHA-256(file) == oczekiwany hash
```

Zwykły checksum nie zawsze potwierdza, kto utworzył plik. Jeśli atakujący może podmienić zarówno plik, jak i opublikowany hash, oba nadal mogą do siebie pasować.

## Podpis cyfrowy

Podpis cyfrowy może potwierdzić:

- integralność,
- autentyczność źródła podpisującego,

pod warunkiem że:

- klucz weryfikacyjny jest zaufany,
- prywatny klucz podpisujący jest chroniony,
- mechanizm został poprawnie wdrożony,
- weryfikacja następuje przed użyciem.

Uproszczony flow:

```text
artefakt podpisany kluczem prywatnym
    -> aplikacja weryfikuje zaufanym kluczem publicznym
    -> akceptacja lub odrzucenie przed użyciem
```

## Szyfrowanie

Szyfrowanie chroni przede wszystkim poufność.

Nie gwarantuje automatycznie integralności ani ochrony przed modyfikacją. Do tego potrzebny jest uwierzytelniony mechanizm, np.:

- MAC lub HMAC,
- podpis cyfrowy,
- authenticated encryption, np. tryb AEAD.

Bezpieczne zdanie:

> Szyfrowanie sprawia, że dane są nieczytelne dla nieuprawnionych osób. Integralność i autentyczność wymagają uwierzytelnionego mechanizmu.

## Replay

Poprawnie podpisana wartość nadal może być niebezpieczna, jeśli można użyć jej ponownie poza właściwym czasem lub kontekstem.

Kontrole mogą obejmować:

- expiry,
- issue time,
- nonce,
- audience,
- powiązanie z akcją,
- powiązanie z sesją,
- jednorazowe użycie,
- unieważnienie po stronie serwera.

## Pytania kontrolne

- Jakiej właściwości potrzebujemy: poufności, integralności, autentyczności czy kilku naraz?
- Skąd pochodzi oczekiwany hash albo klucz weryfikacyjny?
- Czy atakujący może podmienić zarówno artefakt, jak i checksum?
- Czy weryfikacja następuje przed przetwarzaniem?
- Czy błąd powoduje bezpieczne odrzucenie?
- Czy starą, ale poprawnie podpisaną wartość można odtworzyć ponownie?
