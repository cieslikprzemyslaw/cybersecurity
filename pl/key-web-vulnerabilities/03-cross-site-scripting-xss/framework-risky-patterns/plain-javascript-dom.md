# Ryzykowne wzorce XSS w Plain JavaScript i DOM APIs

## Cel

DOM XSS często dzieje się wtedy, gdy JavaScript bierze dane kontrolowane przez atakującego i wpisuje je do strony przez niebezpieczne DOM APIs.

## Typowe źródła danych

Dane kontrolowane przez użytkownika mogą pochodzić z:

```text
location.search
location.hash
location.href
document.referrer
localStorage
sessionStorage
postMessage
API responses
form inputs
```

## Niebezpieczne sinks

Reviewuj szczególnie:

```js
element.innerHTML = userInput;
element.outerHTML = userInput;
document.write(userInput);
element.insertAdjacentHTML("beforeend", userInput);
eval(userInput);
new Function(userInput);
setTimeout(userInput, 1000);
setInterval(userInput, 1000);
```

## Dlaczego to ryzykowne?

Te API mogą traktować stringi jako HTML albo JavaScript. Jeśli attacker-controlled data trafi do jednego z tych sinków, przeglądarka może to wykonać.

## Bezpieczniejsze alternatywy

Dla tekstu:

```js
element.textContent = userInput;
```

Dla atrybutów:

```js
element.setAttribute("title", safeValue);
```

Dla tworzenia DOM:

```js
const p = document.createElement("p");
p.textContent = userInput;
container.appendChild(p);
```

Jeśli HTML musi być renderowany:

- sanitizuj przed renderowaniem,
- używaj ścisłych allowlist,
- blokuj scripts, event handlery, niebezpieczne atrybuty i URL-e,
- unikaj raw HTML, jeśli to możliwe.

## Przykład review

Ryzykowne:

```js
const name = new URLSearchParams(location.search).get("name");
document.getElementById("welcome").innerHTML = name;
```

Bezpieczniej:

```js
const name = new URLSearchParams(location.search).get("name");
document.getElementById("welcome").textContent = name;
```

## Checklist do review

- Jakie jest source danych?
- Czy dane są attacker-controlled?
- Czy trafiają do `innerHTML`, `document.write`, `eval` albo podobnych sinków?
- Czy można użyć `textContent`?
- Czy renderowanie HTML jest naprawdę potrzebne?
- Czy jest sanitizacja, jeśli HTML jest potrzebny?
- Czy URL schemes są walidowane?

## Lekcja dla developera

```text
DOM XSS często jest flow source-to-sink: dane kontrolowane przez atakującego trafiają do niebezpiecznego DOM API.
```
