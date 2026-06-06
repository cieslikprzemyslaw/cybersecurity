# Lekcje OS Command Injection

## Najważniejsze pytanie nie brzmi tylko: czy jest walidacja?

W labie z formularzem feedbacku najpierw podejrzewałem pole `message`, bo free-text fields często mają słabszą walidację. To nie był wystarczająco mocny powód.

Ważniejsze pytanie było:

```text
Która wartość może trafić do API uruchamiania procesu albo komendy shella?
```

Pole `email` było lepszym kandydatem, bo funkcja feedbacku mogła przekazywać adres do procesu związanego z mailem. Nadal była to hipoteza, dopóki timing i output redirection nie potwierdziły wykonania.

## Fakty i założenia muszą być oddzielone

Fakty z blind labu:

- parametr `email` był kontrolowany przez użytkownika,
- separator shella zmienił zachowanie wykonania,
- kontrolowany `ping` spowodował powtarzalne opóźnienie,
- opóźnienie rosło po zwiększeniu liczby pakietów,
- `whoami` wykonało się i zapisało output do pliku,
- plik został pobrany przez endpoint `/image`,
- komenda działała jako niskouprawniony użytkownik labowy `peter-<random-suffix>`.

Założenia:

- oryginalna komenda backendowa prawdopodobnie była związana z obsługą lub wysyłką feedbacku,
- dokładna komenda, executable i shell nie były widoczne,
- cel marketingowy, który początkowo zasugerowałem, nie był potwierdzony dowodami.

## `500` nie dowodzi automatycznie sukcesu ani porażki

Pierwszy timing test wrócił szybko z:

```text
500 Internal Server Error
"Could not save"
```

To nie dowodziło wykonania komendy. Nie dowodziło też, że input jest bezpieczny.

Dopiero separator przed i po wstrzykniętej komendzie dał powtarzalne, proporcjonalne opóźnienie dla `ping -c 5` i `ping -c 10`. Dowodem nie był status HTTP, tylko kontrolowana zmiana czasu.

## Separator po komendzie zmienia strukturę

Lepszy model:

```text
oryginalna komenda ; wstrzyknięta komenda ; dalszy stały tekst
```

Separator po wstrzykniętej komendzie izoluje ją od suffixu, który backend może doklejać po kontrolowanej wartości.

## `>` to operator przekierowania, nie output

Poprawne wyjaśnienie:

```text
whoami > /var/www/images/output.txt
```

`>` przekierowuje stdout z `whoami` do wskazanego pliku.

Endpoint `/image` nie wykonywał komendy. Był tylko osobnym kanałem do pobrania pliku utworzonego przez wstrzykniętą komendę.

## Blind nie znaczy niewidoczne na zawsze

Feedback submission nie zwracał outputu komendy bezpośrednio, więc podatność była blind.

Dowody uzyskano przez:

1. side effect czasowy,
2. side effect w filesystemie,
3. osobny request HTTP pobierający utworzony plik.

## Shell execution i process execution to nie to samo

Process API może uruchomić znany executable bez przekazywania stringa do shella. To mocno ogranicza shell injection, ale nie usuwa automatycznie argument injection.

Ważne pytania:

- Czy shell jest używany?
- Czy executable jest stały?
- Czy argumenty są przekazywane osobno?
- Czy użytkownik może wprowadzić dodatkowe flagi?
- Czy komenda wspiera `--` kończące parsowanie opcji?
- Czy wartości są ściśle allowlistowane?

## Sanitizacja nie jest główną poprawką

Pierwsza odpowiedź remediacyjna zbyt mocno skupiała się na walidacji i sanitizacji.

Lepsza kolejność:

1. nie uruchamiaj komendy OS, jeśli biblioteka albo service API może wykonać zadanie,
2. jeśli process execution jest konieczne, użyj stałego executable i osobnych argumentów bez shella,
3. egzekwuj ścisłe allowlisty i typy dla ograniczonych wartości,
4. uwzględnij argument injection i parsowanie opcji,
5. zastosuj least privilege, ograniczony filesystem, timeouty, limity outputu, logging i monitoring,
6. dodaj testy regresji potwierdzające, że składnia shella i option-like input nie zmieniają wykonania.

## Błędy i korekty

- W labie command injection najpierw wybrałem `message` na podstawie założenia o walidacji, a nie na podstawie prawdopodobnego data flow.
- Początkowo traktowałem odpowiedź `500` jako główny wynik, zamiast oddzielić błąd aplikacji od dowodu wykonania.
- Nauczyłem się używać baseline'u i porównywać kontrolowane zmiany timingu.
- Nauczyłem się, że separator po wstrzykniętej komendzie może być potrzebny do odizolowania jej od suffixu backendu.
- Początkowo opisałem `>` błędnie i poprawiłem to na przekierowanie stdout.
- Pierwsza remediacja zbyt mocno opierała się na sanitizacji. Poprawiony model priorytetyzuje unikanie shella, bezpieczne process APIs, osobne argumenty, allowlisty, least privilege i testy regresji.
