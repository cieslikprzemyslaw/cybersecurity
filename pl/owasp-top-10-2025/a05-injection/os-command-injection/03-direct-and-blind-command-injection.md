# Direct i Blind OS Command Injection

## Direct albo verbose command injection

Direct command injection zwraca output wstrzykniętej komendy w tej samej odpowiedzi HTTP.

Model:

```text
wstrzyknięta komenda wykonuje się
-> stdout trafia do odpowiedzi aplikacji
-> rozpoznawalny output potwierdza wykonanie
```

W prostym labie PortSwigger odpowiedź stock checkera zawierała:

- oczekiwaną wartość stanu magazynowego,
- nazwę użytkownika zwróconą przez `whoami`.

To był bezpośredni dowód, bo output komendy pojawił się w odpowiedzi podatnego endpointu.

## Blind command injection

Blind command injection wykonuje się na serwerze, ale nie zwraca outputu komendy w odpowiedzi podatnego endpointu.

Strona może wyglądać bez zmian, zwrócić generyczny błąd albo zakończyć przetwarzanie asynchronicznie.

Dowód blind opiera się na side effect albo osobnym kanale obserwacji.

## Dowód timingowy

Timing test powinien zacząć się od baseline'u.

Zalecany tok:

1. Wyślij normalny request kilka razy.
2. Zapisz typowy czas odpowiedzi i wariancję.
3. Zmień jeden podejrzany parametr.
4. Poproś o kontrolowane opóźnienie.
5. Powtórz ten sam test.
6. Zmień wartość opóźnienia i sprawdź, czy obserwowany czas zmienia się proporcjonalnie.

Dowód z labu:

```text
normalny request -> milisekundy
ping -c 5       -> około 4,1 sekundy
ping -c 10      -> około 9,3 sekundy
```

W tym labie liczba pakietów dawała mniej więcej sekundę między pakietami, więc opóźnienie było bliskie count minus one. Ważna była nie dokładna formuła, tylko powtarzalność i kontrola proporcji.

Jeden wolny request nie jest dowodem, bo opóźnienie sieci, obciążenie serwera, retry albo niezwiązana awaria mogą dać false positive.

## Dowód plikowy

Jeśli stdout jest ukryty, ale istnieje zapisywalna i możliwa do pobrania lokalizacja, output można przekierować w autoryzowanym labie.

Przykład koncepcyjny:

```text
whoami > /var/www/images/output.txt
```

`>` przekierowuje standard output do pliku.

Potem osobny request może pobrać plik:

```http
GET /image?filename=output.txt
```

Warunki:

- proces aplikacji może zapisać wybrany katalog,
- endpoint pobierania mapuje się na ten katalog,
- komenda kończy działanie,
- plik outputu jest dostępny,
- cleanup nie usuwa pliku wcześniej.

Endpoint plikowy jest kanałem obserwacji. Nie musi być sinkiem command execution.

## Dowód out-of-band

Gdy komenda wykonuje się asynchronicznie albo output i pliki są niedostępne, autoryzowany serwis OAST może zaobserwować interakcję outbound.

Kanały koncepcyjne:

- DNS lookup,
- HTTP request,
- inna kontrolowana interakcja sieciowa.

Dowód OAST powinien używać unikalnego markera, aby połączyć interakcję z jednym requestem testowym.

## Różnice statusu i odpowiedzi

Zmiana statusu albo body może być użyteczną wskazówką, ale nie jest automatycznie dowodem wykonania.

Przykład z blind labu:

```text
500 Internal Server Error
"Could not save"
```

Ten sam błąd pojawiał się przy requestach szybkich i opóźnionych. `500` pokazywał, że input zmienił zachowanie aplikacji, ale timing i odczyt pliku były właściwym dowodem execution.

## Separatory komend i izolacja

Separatory mogą zmienić jeden string komendy w wiele komend.

| Operator | Ogólne zachowanie |
|---|---|
| `;` | wykonaj kolejną komendę niezależnie od poprzedniej w shellach Unix-like |
| `&&` | wykonaj kolejną komendę tylko po sukcesie poprzedniej |
| `||` | wykonaj kolejną komendę tylko po porażce poprzedniej |
| `&` | separator albo operator tła, zależnie od shella i kontekstu |
| `|` | przekaż stdout jednej komendy do drugiej |
| newline | separator komend w shellach Unix-like |

Separator po wstrzykniętej komendzie może być ważny:

```text
oryginalna komenda ; wstrzyknięta komenda ; oryginalny suffix
```

Bez niego stały suffix może stać się argumentami wstrzykniętej komendy albo spowodować błąd składni.

Macierz PortSwigger traktuje `;` i newline jako separatory Unix-only, a `&`, `&&`, `|` i `||` jako typowe dla kontekstów Windows i Unix. PowerShell ma własne reguły i trzeba go oceniać osobno.

## Kontekst cudzysłowów

Kontrolowany input może znaleźć się w cudzysłowie:

```text
command "USER_INPUT" fixed-suffix
```

Wtedy składnia komendy może pozostać wewnątrz cytowanego argumentu, dopóki kontekst nie zostanie zamknięty. Dlatego losowe payload guessing jest słabe. Lepiej testować jedną hipotezę o kontekście naraz.

## Encoding i transport

W `application/x-www-form-urlencoded`:

- `%3B` dekoduje się do `;`,
- `+` dekoduje się do spacji,
- niezakodowany `&` zaczyna nowy parametr formularza,
- `%26` oznacza ampersand wewnątrz wartości.

Poprawny transport encoding zapewnia tylko, że backend dostanie zamierzone znaki. Nie czyni wartości bezpieczną.

## Hierarchia dowodów

Od słabszych do mocniejszych:

1. generyczny błąd albo zmiana statusu,
2. powtarzalna różnica odpowiedzi związana z jedną hipotezą,
3. powtarzalna zmiana timingu,
4. proporcjonalna kontrola timingu,
5. unikalny side effect w pliku albo aplikacji,
6. bezpośredni output komendy,
7. unikalna interakcja OAST powiązana z requestem.

Najmocniejszy dowód jest konkretny, powtarzalny, kontrolowany i trudny do wyjaśnienia normalnym zachowaniem aplikacji.

## Takeaway

Direct command injection zwraca output komendy w tej samej odpowiedzi. Blind command injection wymaga wiarygodnego side channel, np. timingu, pliku albo interakcji outbound. Generyczny błąd może być wskazówką, ale dowód powinien pokazywać zmianę OS execution, nie tylko nieudaną walidację.
