# Ryzykowne wzorce XSS w PHP

## Cel

PHP często renderuje HTML po stronie serwera. XSS może pojawić się, gdy dane kontrolowane przez użytkownika są wypisywane do strony bez context-aware escaping.

## Ryzykowny wzorzec: bezpośrednie echo user input

```php
<h1>Hello, <?php echo $_GET['name']; ?></h1>
```

## Dlaczego to ryzykowne?

Jeśli użytkownik kontroluje `name`, może wstrzyknąć HTML albo JavaScript do response.

Przykładowy output:

```html
<h1>Hello, <script>alert(1)</script></h1>
```

Przeglądarka może wykonać skrypt.

## Bezpieczniejsza alternatywa: HTML escaping

```php
<h1>Hello, <?php echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8'); ?></h1>
```

## Dlaczego to pomaga?

`htmlspecialchars` zamienia specjalne znaki HTML na bezpieczne reprezentacje tekstowe.

Na przykład:

```text
< staje się &lt;
> staje się &gt;
" staje się &quot;
' staje się &#039; przy ENT_QUOTES
```

Dzięki temu przeglądarka pokazuje input jako tekst, zamiast interpretować go jako HTML.

## Uwaga na attribute context

HTML body context i attribute context to nie to samo.

Ryzykowne:

```php
<input value="<?php echo $_GET['name']; ?>">
```

Bezpieczniej:

```php
<input value="<?php echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8'); ?>">
```

`ENT_QUOTES` jest ważne, bo cudzysłowy/apostrofy mogą pozwolić wyjść z atrybutu.

## Uwaga na JavaScript context

Nie wkładaj bezpośrednio user input do JavaScriptu.

Ryzykowne:

```php
<script>
  const name = "<?php echo $_GET['name']; ?>";
</script>
```

Bezpieczniejsze podejścia:

- unikaj inline JavaScript z user input,
- przekazuj dane jako JSON używając bezpiecznego encodingu,
- używaj bezpiecznych mechanizmów frameworka,
- stosuj encoding dopasowany do kontekstu.

## Checklist do review

- Czy `$_GET`, `$_POST`, `$_REQUEST` albo dane z bazy są echo bez escaping?
- Czy output jest escapowany przez `htmlspecialchars`?
- Czy `ENT_QUOTES` jest użyte w attribute context?
- Czy określono UTF-8?
- Czy user input jest wkładany do JavaScriptu?
- Czy raw HTML jest naprawdę potrzebny?
- Czy autoescape template jest włączony?

## Developer takeaway

```text
W PHP bezpośrednie echo user-controlled data bez escaping to częsty risk XSS.
```

Używaj context-aware escaping i unikaj raw output, chyba że dane są zaufane i sanitizowane.
