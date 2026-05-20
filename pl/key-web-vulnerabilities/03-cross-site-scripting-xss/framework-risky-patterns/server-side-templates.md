# Ryzykowne wzorce XSS w server-side templates

## Cel

Server-side templates generują HTML po stronie backendu. XSS może wystąpić, gdy dane kontrolowane przez użytkownika są renderowane jako raw HTML zamiast escapowanego tekstu.

## Czym są server-side templates?

Server-side templates łączą HTML z danymi.

Przykłady:

```text
PHP templates
Twig
Blade
Jinja2
Django templates
EJS
Handlebars
Pug
Razor
Thymeleaf
```

Backend produkuje gotowy HTML response i wysyła go do przeglądarki.

## Typowy ryzykowny wzorzec

Wiele template engine’ów domyślnie escapuje wartości, ale daje sposób na renderowanie raw HTML.

Reviewuj te patterny:

```text
PHP: echo $userInput bez htmlspecialchars
Twig: {{ value|raw }}
Jinja / Django: {{ value|safe }}
EJS: <%- value %>
Handlebars: {{{ value }}}
Razor: @Html.Raw(value)
Thymeleaf: th:utext
```

## Dlaczego to ryzykowne?

Raw output mówi template engine’owi, żeby nie escapował HTML. Jeśli wartość jest user-controlled, przeglądarka może wykonać wstrzyknięty markup albo JavaScript.

## Bezpieczniejsze alternatywy

Używaj escapowanego outputu domyślnie.

Przykłady:

```text
Twig: {{ value }}
Jinja / Django: {{ value }}
EJS: <%= value %>
Handlebars: {{ value }}
Razor: @value
Thymeleaf: th:text
```

Jeśli raw HTML jest wymagany:

- sanitizuj przed renderowaniem,
- używaj ścisłej allowlisty,
- blokuj scripts i event handlery,
- waliduj URL-e,
- dokumentuj, dlaczego raw output jest potrzebny.

## Kontekst ma znaczenie

Escaping musi pasować do kontekstu:

```text
HTML body
HTML attribute
JavaScript string
URL
CSS
```

Escaping dla jednego kontekstu może nie być bezpieczny dla innego.

## Checklist do review

- Czy autoescaping jest włączony?
- Czy użyto raw output?
- Skąd pochodzi raw value?
- Czy wartość jest user-controlled albo CMS-controlled?
- Czy jest sanitizacja?
- Czy output trafia do HTML body, attribute, JavaScript, URL albo CSS?
- Czy można użyć normalnego escaped output?

## Developer takeaway

```text
Autoescaping pomaga, ale raw output features omijają zabezpieczenia template engine.
```

Każda funkcja raw output powinna być punktem security review.
