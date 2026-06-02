# Typy SQL Injection

SQL Injection można podzielić na kilka głównych typów. Zrozumienie typu ma znaczenie, bo każdy ma inny wzorzec dowodów.

## Główne typy SQLi

```text
SQL Injection
├── In-band SQLi
│   ├── Error-based SQLi
│   └── Union-based SQLi
├── Inferential / Blind SQLi
│   ├── Boolean-based Blind SQLi
│   └── Time-based Blind SQLi
└── Out-of-band SQLi
```

## In-band SQL Injection

In-band SQLi oznacza, że ten sam kanał komunikacji jest używany do ataku i odbioru wyniku.

### Error-based SQLi

Atakujący powoduje błędy bazy danych i używa komunikatów błędów do poznania struktury bazy, zachowania zapytania albo typu bazy.

Dowód:

```text
Komunikaty błędów bazy pojawiają się w odpowiedzi.
```

### Union-based SQLi

Atakujący używa `UNION SELECT`, aby dołączyć wynik innego zapytania SELECT do normalnej odpowiedzi aplikacji.

Dowód:

```text
Dane z innej tabeli pojawiają się w odpowiedzi webowej.
```

## Inferential / Blind SQL Injection

Blind SQLi nie zwraca danych z bazy bezpośrednio. Atakujący wnioskuje informacje z zachowania aplikacji.

### Boolean-based blind SQLi

Atakujący wysyła warunki SQL true i false, a potem obserwuje, czy odpowiedź się zmienia.

Dowód:

```text
Warunek TRUE -> jedno zachowanie odpowiedzi
Warunek FALSE -> inne zachowanie odpowiedzi
```

### Time-based blind SQLi

Atakujący wysyła warunek, który powoduje opóźnienie, jeśli jest prawdziwy.

Dowód:

```text
Odpowiedź jest opóźniona tylko wtedy, gdy wstrzyknięty warunek jest prawdziwy.
```

## Out-of-band SQL Injection

Out-of-band SQLi używa osobnego kanału dla wyniku albo dowodu, na przykład DNS, HTTP albo SMB.

Dowód:

```text
Zewnętrzny callback, zewnętrzny request albo plik zapisany w lokalizacji kontrolowanej przez testera.
```

## Notatka AppSec

Typ SQLi zmienia sposób zbierania dowodów, ale przyczyna źródłowa jest ta sama: niebezpieczne budowanie zapytań SQL z niezaufanego inputu.
