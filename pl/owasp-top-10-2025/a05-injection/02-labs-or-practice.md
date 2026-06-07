# A05: Injection - Laby i praktyka

Ta kategoria zawiera obecnie ukończoną praktykę dla SQL Injection, NoSQL Injection, OS Command Injection i Server-Side Template Injection.

Wszystkie opisane testy praktyczne były wykonywane wyłącznie w TryHackMe, PortSwigger Web Security Academy, lokalnych labach albo innym jawnie autoryzowanym środowisku.

## Ukończone laby SQL Injection

- [UNION-based SQL Injection - retrieving data from other tables](sql-injection/labs/01-union-retrieve-data-from-other-tables.md)
- [Blind SQL Injection with conditional responses](sql-injection/labs/02-blind-conditional-responses.md)

## Ukończona praktyka NoSQL Injection

### TryHackMe

- NoSQL Injection room
- podstawy dokumentów i kolekcji MongoDB,
- filtry zapytań MongoDB i zagnieżdżone operatory,
- obejście uwierzytelniania przez Operator Injection,
- ekstrakcja true/false przy użyciu `$regex`,
- świadomość Syntax Injection w custom JavaScript-style queries.

### PortSwigger Web Security Academy

- [Detecting NoSQL injection](nosql-injection/labs/01-detecting-nosql-injection.md)
- [Exploiting NoSQL injection to extract data](nosql-injection/labs/02-extracting-data-with-a-boolean-oracle.md)
- [Porównanie labów NoSQL](nosql-injection/labs/summary.md)

## Ukończona praktyka OS Command Injection

### TryHackMe

- teoria OS Command Injection,
- przykład PHP z `exec()` i tytułem piosenki kontrolowanym przez użytkownika,
- przykład Python Flask z `subprocess.Popen(..., shell=True)`,
- direct/verbose oraz blind command injection,
- timing, output redirection i ogólne dowody out-of-band,
- różnice między komendami Linux i Windows,
- priorytety defensywne oraz ograniczenia samej sanitizacji.

### PortSwigger Web Security Academy

- [Simple OS command injection](os-command-injection/labs/01-simple-command-injection.md)
- [Blind OS command injection z output redirection](os-command-injection/labs/02-blind-command-injection-output-redirection.md)
- [Porównanie labów OS Command Injection](os-command-injection/labs/summary.md)

## Ukończona praktyka Server-Side Template Injection

### TryHackMe

- teoria SSTI i praktyka dla kilku template engine,
- Smarty / PHP: ewaluacja wyrażeń i świadomość security policy,
- Pug / Node.js: ewaluacja wyrażeń i lekcje z process API,
- Jinja2 / Python: object traversal i lekcje z `subprocess`,
- świadomość SSTImap jako narzędzia do autoryzowanych labów.

### PortSwigger Web Security Academy

- [Basic server-side template injection using ERB](server-side-template-injection/labs/01-basic-ssti-erb.md)
- [Porównanie praktyki SSTI](server-side-template-injection/labs/summary.md)

## Pokrycie praktyczne

Ukończona praktyka obejmuje:

- identyfikowanie danych kontrolowanych przez użytkownika, które trafiają do interpretera albo mechanizmu wykonania,
- odróżnianie danych od wykonywalnej składni,
- rozpoznawanie, kiedy backend może budować pełną komendę shella,
- oddzielanie faktów od założeń o oryginalnej komendzie backendowej,
- używanie baseline'u przed interpretacją błędów lub zmian odpowiedzi,
- używanie outputu komendy jako bezpośredniego dowodu,
- używanie powtarzalnego i proporcjonalnego timingu jako dowodu blind,
- używanie side effectu w pliku i osobnego endpointu do odzyskania ukrytego outputu,
- rozumienie, dlaczego odpowiedź `500` nie dowodzi automatycznie sukcesu ani porażki wykonania,
- odróżnianie shell injection od argument injection,
- odróżnianie odbicia tekstu od server-side template evaluation,
- fingerprinting template engine na podstawie nieszkodliwych, kontrolowanych dowodów,
- oddzielanie root cause SSTI od głębszych skutków, takich jak dostęp do plików albo uruchamianie procesów,
- używanie Burp Intruder dopiero po ręcznym potwierdzeniu oracle,
- przekładanie dowodów technicznych na remediację i testy regresji.

## Aktualne ograniczenia

Ten katalog A05 nie zawiera jeszcze ukończonych modułów praktycznych dla:

- AI Prompt Injection,
- mapowania XSS pod A05.

Te tematy powinny zostać dodane dopiero po ukończeniu odpowiedniej nauki i praktyki.
