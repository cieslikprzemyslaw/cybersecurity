# Gdzie Testować Path Traversal / File Inclusion

## Cel

Używaj tej checklisty, gdy szukasz miejsc, w których input użytkownika może wpływać na dostęp do plików, downloady, template'y, obrazki, pliki językowe albo include'y.

Najważniejsze praktyczne pytanie:

```txt
Gdzie input użytkownika może wpływać na to, jaki plik backend czyta, ładuje albo dołącza?
```

## Parametry wysokiego sygnału

| Parametr | Dlaczego jest ciekawy | Przykład |
|---|---|---|
| `file` | Często wybiera plik | `?file=report.pdf` |
| `filename` | Często używany przy obrazkach/downloadach | `?filename=avatar.jpg` |
| `path` | Może bezpośrednio kontrolować ścieżkę | `?path=/files/doc.pdf` |
| `page` | Często używany w PHP include/nawigacji | `?page=about.php` |
| `lang` | Może ładować pliki językowe | `?lang=en` |
| `template` | Może ładować server-side templates | `?template=home` |
| `view` | Może wybierać view/template | `?view=profile` |
| `document` | Może zwracać PDF/dokument | `?document=invoice.pdf` |
| `download` | Może uruchamiać pobieranie pliku | `?download=contract.pdf` |
| `image` / `img` | Może pobierać obrazki z dysku | `?image=1.jpg` |
| `avatar` | Może pobierać avatary użytkowników | `?avatar=user.png` |

## Miejsca, gdzie może trafić payload

Payload nie zawsze idzie w query string.

| Miejsce | Przykład | Notatka |
|---|---|---|
| Query string | `GET /image?filename=1.jpg` | Najczęstsze w labach |
| POST body | `file=report.pdf` | Częste w formularzach/downloadach |
| Cookies | `Cookie: file=admin` | Możliwe przy PHP `$_REQUEST` albo custom logice |
| Headers | `X-File: report.pdf` | Mniej częste, ale możliwe w custom API |
| JSON body | `{"file":"report.pdf"}` | Częste w nowoczesnych API |
| Hidden fields | `<input type="hidden" name="file">` | Często pomijane w formularzach |
| Route params | `/download/report.pdf` | Ścieżka może być częścią route'a |

## Czy search box to dobre miejsce do testowania?

Zwykle nie.

Normalny search input typu:

```http
GET /search?q=test
```

zwykle trafia do wyszukiwarki albo zapytania do bazy danych, nie do filesystemu.

Search robi się interesujący tylko wtedy, gdy wartość wygląda jak wybór pliku, template'u, raportu, exportu, strony, dokumentu albo zasobu do pobrania.

## Sygnały w request/response

Szukaj endpointów, które ładują albo zwracają:

- obrazki,
- avatary,
- downloady,
- preview PDF/dokumentów,
- załączniki,
- pliki językowe,
- template'y/views,
- raporty/exporty.

Ciekawe response'y:

- `"No such file"`,
- `"File not found"`,
- PHP include warnings,
- pełne ścieżki filesystemu w błędach,
- `Content-Type` mówi image, ale body zawiera tekst,
- inna długość response po payloadzie,
- output podobny do `/etc/passwd`.

## Podstawowy flow testowania

1. Znajdź request, który ładuje plik.

```http
GET /image?filename=4.jpg
```

2. Zmień wartość na losowy string.

```http
GET /image?filename=test
```

3. Obserwuj response.

Szukaj błędów typu `"No such file"` albo zmian w długości response.

4. Testuj basic traversal.

```http
GET /image?filename=../../../etc/passwd
```

5. Jeśli zablokowane, testuj alternatywne metody.

- Absolute path.
- Nested traversal.
- URL encoding.
- Double URL encoding.
- Required base path bypass.

6. Potwierdź, czy response zawiera zawartość pliku.

7. Zanotuj, która walidacja albo mapowanie plików zawiodło.

## Najważniejsze przypomnienie

Nie testuj losowych inputów wszędzie. Najpierw zidentyfikuj miejsca, gdzie input użytkownika wygląda, jakby wpływał na server-side file access.
