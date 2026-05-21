# CSRF + SameSite Cookies

## TL;DR

Cross-Site Request Forgery (CSRF) to podatność, w której atakujący nakłania przeglądarkę zalogowanego użytkownika do wysłania żądania do aplikacji, w której użytkownik ma aktywną sesję.

Atakujący nie musi kraść hasła ani session cookie. Przeglądarka może automatycznie dołączyć cookie sesyjne ofiary do requestu wysłanego do docelowej aplikacji. Jeśli aplikacja ufa samemu cookie, akcja zmieniająca stan może zostać wykonana bez świadomej intencji użytkownika.

## Co ćwiczyłem

Ćwiczyłem CSRF w labach TryHackMe i PortSwigger.

Zakres praktyki:

- szukanie requestów zmieniających stan,
- sprawdzanie, czy request opiera się tylko na session cookie,
- rozpoznawanie obecności lub braku CSRF tokena,
- testowanie brakującej lub słabej ochrony CSRF,
- zrozumienie, dlaczego `POST` nie chroni automatycznie przed CSRF,
- sprawdzanie, czy token jest powiązany z sesją użytkownika,
- analiza działania SameSite cookies,
- testowanie obejścia `SameSite=Lax` przez method override,
- zrozumienie, dlaczego CORS, Referer i custom headers nie są pełną ochroną same w sobie.

## Kluczowa koncepcja

CSRF występuje wtedy, gdy serwer traktuje request jako intencjonalny tylko dlatego, że zawiera poprawne session cookie.

Cookie sesyjne potwierdza, że przeglądarka ma aktywną sesję. Nie potwierdza, że użytkownik świadomie wykonał daną akcję.

Typowy atak CSRF wymaga:

- zalogowanej ofiary,
- akcji zmieniającej stan,
- braku poprawnej walidacji pochodzenia requestu lub intencji użytkownika.

Typowe funkcje podatne na CSRF:

- zmiana adresu email,
- zmiana hasła,
- aktualizacja ustawień konta,
- edycja profilu,
- zmiana danych płatniczych,
- zapis preferencji użytkownika,
- zmiana roli lub uprawnień.

`POST` nie jest zabezpieczeniem przed CSRF. Złośliwa strona nadal może wysłać `POST` przez formularz HTML. `GET` jest szczególnie ryzykowny, jeśli zmienia stan, ponieważ może być wywołany przez link, przekierowanie albo obrazek.

CSRF token pomaga tylko wtedy, gdy jest silny, nieprzewidywalny, walidowany po stronie serwera i powiązany z sesją użytkownika.

## Co kontroluje użytkownik

W labach atakujący kontrolował:

- pola formularza, np. `email`,
- parametry w URL,
- ukryte pola formularza, np. `csrf`,
- formularz HTML hostowany na exploit serverze,
- JavaScript używany do automatycznego wysłania formularza lub przekierowania,
- parametr `_method` używany do method override,
- w niektórych scenariuszach przewidywalny albo wielokrotnego użycia CSRF token.

Atakujący nie kontrolował bezpośrednio session cookie ofiary. Cookie zostało automatycznie dołączone przez przeglądarkę, kiedy request trafił do docelowej aplikacji.

## Where the Input Goes

Kontrolowane dane trafiały do backendowych endpointów zmieniających stan, głównie:

- `/my-account/change-email`,
- funkcji zmiany hasła w ćwiczeniach CSRF,
- funkcji zmiany roli lub ustawień konta,
- walidacji CSRF tokena po stronie serwera,
- mechanizmu method override, który zamieniał top-level `GET` na backendowe zachowanie `POST`.

W labie SameSite ważny był schemat:

```http
GET /my-account/change-email?email=...&_method=POST
```

Przeglądarka widziała request jako `GET`, a backend traktował go jako akcję typu `POST`.

## Wpływ

Wpływ CSRF zależy od tego, co robi podatna funkcja.

Realistyczny impact:

- zmiana adresu email użytkownika,
- zmiana hasła,
- aktualizacja ustawień konta,
- zmiana danych płatniczych,
- modyfikacja roli,
- wykonanie niechcianej akcji jako ofiara.

CSRF zwykle nie pozwala atakującemu odczytać odpowiedzi, ponieważ przeglądarka ma mechanizmy ochronne takie jak Same-Origin Policy. Odczyt odpowiedzi nie zawsze jest jednak potrzebny, jeśli sama akcja została wykonana.

## Remediacja dla developerów

Rekomendowane poprawki:

- dodawać silne, losowe i nieprzewidywalne CSRF tokeny do requestów zmieniających stan,
- wiązać CSRF token z sesją użytkownika,
- walidować token po stronie serwera przy każdej wrażliwej akcji,
- odrzucać brakujące, puste, użyte ponownie, przewidywalne lub cross-session tokeny,
- nie generować tokenów z przewidywalnych danych, takich jak numer konta, rola, username lub Base64,
- unikać akcji zmieniających stan przez `GET`,
- nie traktować `POST` jako zabezpieczenia,
- unikać niebezpiecznego method override dla wrażliwych akcji,
- ustawiać session cookies z odpowiednią wartością `SameSite`,
- używać `SameSite=Lax` lub `SameSite=Strict`, gdy jest to możliwe,
- używać `SameSite=None; Secure` tylko wtedy, gdy cross-site cookies są naprawdę wymagane,
- walidować `Origin` i/lub `Referer` jako defence-in-depth, nie jako jedyną ochronę,
- ostrożnie konfigurować CORS i nie dopuszczać credentialed requests z niezaufanych originów,
- naprawiać XSS, bo XSS może często obejść ochronę CSRF przez odczyt tokena lub wykonanie same-origin requestu.

## Review Checklist

Podczas testowania lub code review zapytaj:

- Czy ten request zmienia stan po stronie serwera?
- Czy request opiera się tylko na session cookie?
- Czy istnieje CSRF token?
- Czy token jest losowy i nieprzewidywalny?
- Czy token jest powiązany z sesją użytkownika?
- Co się stanie, jeśli token zostanie usunięty?
- Co się stanie, jeśli użyję tokena z innego użytkownika?
- Czy świeży token z jednego konta działa z sesją innego konta?
- Czy request można odtworzyć z zewnętrznego formularza HTML?
- Czy akcja zmieniająca stan działa przez `GET`?
- Czy aplikacja obsługuje `_method=POST` albo podobny method override?
- Czy method override pozwala obejść założenia `SameSite=Lax`?
- Jakie SameSite ma session cookie?
- Czy CORS jest bezpieczny dla wrażliwych endpointów?
- Czy aplikacja polega tylko na Referer lub custom headerach?

## Główna lekcja

CSRF polega na nadużyciu normalnego zachowania przeglądarki, a nie na kradzieży cookie.

Poprawne session cookie potwierdza uwierzytelnienie, ale nie potwierdza intencji użytkownika.

Najważniejsza lekcja:

> Ochrona przed CSRF musi sprawdzać, czy request został świadomie wykonany przez zalogowanego użytkownika, a nie tylko czy przeglądarka wysłała poprawne session cookie.
