# Lab: Exploiting NoSQL Injection to Extract Data

> PortSwigger Web Security Academy: Exploiting NoSQL injection to extract data

## Cel

Użyć NoSQL Syntax Injection do odzyskania hasła administratora przez boolean oracle w legalnym labie.

## Ważna korekta procesu

Widoczna strona konta nie była właściwym miejscem testu:

```text
/my-account?id=wiener
```

Podatny request znajdował się w ruchu w tle:

```text
/user/lookup?user=wiener
```

Ten request był widoczny w Burp HTTP history.

## Baseline

Normalny lookup zwracał dane użytkownika.

Trzeba było ustalić:

- jak wygląda odpowiedź normalna,
- jaki marker oznacza true,
- jaki marker oznacza false,
- jak endpoint reaguje na błędną składnię.

## Potwierdzenie Syntax Injection

Apostrof powodował błąd albo zmianę odpowiedzi, co sugerowało wpływ na składnię query.

Następnie kontrolowane warunki true/false potwierdziły, że input może zmieniać logikę query.

## Boolean oracle

Endpoint nie zwracał hasła bezpośrednio. Zwracał różne odpowiedzi zależnie od tego, czy warunek był prawdziwy.

Model:

```text
warunek o haśle jest prawdziwy -> marker true
warunek o haśle jest fałszywy  -> marker false
```

## Ustalenie długości hasła

Najpierw testowana była długość hasła administratora.

Koncepcyjnie:

```text
this.password.length == 1
this.password.length == 2
this.password.length == 3
```

Poprawna długość dawała marker true.

## Ekstrakcja znaków

Potem testowana była każda pozycja i każdy kandydat znaku.

Koncepcyjnie:

```text
this.password[0] == "a"
this.password[0] == "b"
this.password[1] == "a"
```

Dla każdej pozycji jeden znak dawał marker true.

## Burp Intruder

Automatyzacja była użyteczna dopiero po ręcznym potwierdzeniu oracle.

Cluster Bomb pasował do testowania wszystkich kombinacji:

- pozycji hasła,
- kandydatów znaków.

Pitchfork nie byłby właściwy dla pełnej kombinacji, bo paruje payloady z list.

## Najważniejsze lekcje

- Nie zakładać, że podatny endpoint to `/login`.
- Szukać background API requests.
- Najpierw potwierdzić oracle ręcznie.
- Dbać o poprawny encoding.
- Rozumieć, że `this.password[0]` oznacza pierwszy znak stringa JavaScript.

## Remediacja

- Usunąć budowanie custom JavaScript query.
- Nie doklejać inputu do `$where`.
- Egzekwować, że `user` jest stringiem.
- Nie przechowywać ani nie odpytywać plaintext passwords.
- Normalizować odpowiedzi zależne od sekretu.
- Dodać rate limiting i monitoring prób ekstrakcji.
