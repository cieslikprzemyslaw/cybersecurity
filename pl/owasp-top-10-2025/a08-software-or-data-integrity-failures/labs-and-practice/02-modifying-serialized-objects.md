# PortSwigger - Modifying Serialized Objects

## Źródło

[Lab: Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)

## Cel

Sprawdzić, czy backend ufa właściwości istotnej z punktu widzenia bezpieczeństwa zapisanej w serializowanym obiekcie sesji kontrolowanym przez klienta.

## Oryginalny stan

Session cookie było zakodowane przez URL i Base64.

Po odkodowaniu:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

Interpretacja:

```text
User {
    username = "wiener"
    admin = false
}
```

## Testowane założenie integralności

> Backend może ufać właściwości `admin` z obiektu sesji kontrolowanego przez klienta bez weryfikacji integralności albo stanu autoryzacji po stronie serwera.

## Kontrolowana zmiana

Zmieniłem wyłącznie:

```text
b:0
```

na:

```text
b:1
```

Zmieniony obiekt:

```text
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

Następnie obiekt został ponownie zakodowany przez Base64 i URL encoding.

## Dowody

### Dowód 1: zachowanie UI

Response pokazał:

```text
/admin
```

Był to użyteczny, ale niewystarczający dowód, ponieważ widoczny link sam nie potwierdza autoryzacji endpointu.

### Dowód 2: chroniony endpoint

Request:

```http
GET /admin
```

zwrócił panel administratora z akcjami usuwania użytkowników.

To potwierdziło administrative access.

### Dowód 3: akcja administracyjna

Request:

```http
GET /admin/delete?username=carlos
```

zwrócił:

```http
HTTP/2 302 Found
Location: /admin
```

Akcja została wykonana i lab został ukończony.

## Fakty

- Session cookie zawierało serializowany obiekt PHP.
- Obiekt zawierał kontrolowany przez klienta boolean `admin`.
- Base64 i URL encoding nie zapobiegały modyfikacji.
- Backend zaakceptował `admin=true`.
- Backend zwrócił panel administratora.
- Backend pozwolił usunąć innego użytkownika.

## Założenia, których nie potrzebowaliśmy

- Brak deklaracji remote code execution.
- Brak deklaracji istnienia gadget chain.
- Brak deklaracji, że wszystkie sesje aplikacji używały tego mechanizmu.
- Brak deklaracji dotyczącej dokładnego kodu backendu.

## Integrity failure

Aplikacja zaakceptowała zmieniony serializowany stan klienta bez wystarczającej weryfikacji integralności.

## Access-control impact

Zaakceptowany stan spowodował privilege escalation:

```text
zwykły użytkownik
    -> zmieniona właściwość admin
    -> administrative access
    -> usunięcie innego użytkownika
```

## Root cause

> Aplikacja deserializowała dane sesji kontrolowane przez klienta i ufała właściwości `admin` istotnej z punktu widzenia bezpieczeństwa bez wystarczającej ochrony integralności lub weryfikacji autoryzacji po stronie serwera.

## Bezpieczny projekt

Preferowane:

```text
opaque session identifier
    -> server-side session store
    -> rola pobrana z zaufanych danych
    -> autoryzacja na każdym endpoincie
```

Jeśli stan klienta jest konieczny:

- chronić go MAC lub podpisem,
- weryfikować przed deserializacją albo użyciem,
- powiązać z kontekstem i expiry,
- odrzucać niepoprawny stan,
- nadal egzekwować autoryzację po stronie serwera,
- unikać generic native object deserialization.

## Test regresyjny

Dla poprawnej sesji non-admin zmiana client-side admin value nie może:

- ustawić zaufanego stanu admin,
- pozwolić na `/admin`,
- umożliwić akcji administracyjnych.

Request powinien zostać odrzucony albo oceniony na podstawie zaufanych server-side permissions.
