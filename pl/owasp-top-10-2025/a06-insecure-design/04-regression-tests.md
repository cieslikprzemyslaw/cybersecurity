# A06: Insecure Design - Pomysły na testy regresji

Testy regresji powinny dowodzić, że wymagana reguła biznesowa pozostaje prawdziwa, gdy klient modyfikuje wartości, omija UI albo używa workflow w nieoczekiwanej kolejności.

## Autorytatywne wartości kontrolowane przez klienta

- Zmienić przesłaną cenę produktu; serwer musi ją zignorować albo odrzucić i obliczyć kwotę z zaufanych danych produktowych.
- Usunąć pole ceny; serwer nadal musi obliczyć poprawną kwotę.
- Zmienić role, owner ID, user ID, event ID i booking ID; nieautoryzowane akcje muszą zwrócić `403` i nie zmienić stanu.
- Zmienić pola statusu, takie jak `published`, `paid` lub `confirmed`; klient nie może wybierać autorytatywnego stanu.
- Sprawdzić, czy API nie zwraca wrażliwych pól jedynie ukrytych przez frontend.

## Pomijanie, powtarzanie i zmiana kolejności

- Pominąć etap płatności i spróbować potwierdzić rezerwację.
- Powtórzyć kupon po zastosowaniu innego prawidłowego kuponu.
- Replayować wykorzystany token resetu hasła.
- Powtórzyć request confirmation, cancellation, refund i notification.
- Zmienić kolejność płatności, potwierdzenia i wydania biletu.
- Spróbować reaktywować rekord wygasły lub anulowany.
- Powtórzyć finalny request po poprawnym zakończeniu operacji.

## Testy przejść stanu

- Potwierdzić, że akceptowane są tylko udokumentowane przejścia.
- Odrzucić nieznane source i destination states.
- Odrzucić `PENDING_PAYMENT -> CONFIRMED` bez zaufanego dowodu płatności.
- Unieważnić bilet po cancellation albo refund.
- Nie pozwolić, aby zrefundowana rezerwacja wróciła do paid lub active bez zatwierdzonego recovery flow.
- Ponownie sprawdzić autorytatywny bieżący stan bezpośrednio przed zmianą.

## Autoryzacja i ownership

- Użyć dwóch użytkowników i zamienić resource identifiers.
- Osobno przetestować read, update, delete, download, export i akcje administracyjne.
- Sprawdzić trasy normal, alternate, legacy, batch i direct-service.
- Przetestować każdą rolę dla tej samej wrażliwej akcji.
- Potwierdzić, że odmowa nie zwraca prywatnych danych i nie zmienia stanu.
- Potwierdzić, że prywatne dane są filtrowane przed budowaniem odpowiedzi listing i search.

## Sesje i odzyskiwanie konta

- Użyć dwóch przeglądarek i sprawdzić starą sesję po logout.
- Powtórzyć po password reset, password change, account disablement i role change.
- Sprawdzić tokeny zmodyfikowane, wygasłe, wykorzystane i przypisane do innego konta.
- Porównać odpowiedzi login, registration i recovery dla istniejących i nieistniejących kont.
- Sprawdzić, czy status, body, długość, redirect i timing nie tworzą wiarygodnego enumeration oracle.

## Concurrency i idempotency

- Wysłać wiele żądań rezerwacji ostatniego wolnego miejsca.
- Wysłać duplikaty payment i refund równolegle.
- Ponownie użyć tego samego idempotency key i sprawdzić, czy zwracany jest pierwotny wynik.
- Użyć różnych keys dla tej samej operacji biznesowej i potwierdzić, że constraints domeny nadal zapobiegają duplikatom.
- Potwierdzić, że capacity, balance i state nigdy nie stają się ujemne albo sprzeczne.

## Webhooki i wiadomości zewnętrzne

- Wysłać webhook bez podpisu i z błędnym podpisem.
- Zmienić podpisany payload.
- Replayować wcześniej prawidłowy event.
- Wysłać event wygasły albo w złej kolejności.
- Użyć niewłaściwego event type, amount, currency, booking lub provider reference.
- Potwierdzić, że żaden stan wewnętrzny nie zmienia się przed weryfikacją autentyczności, świeżości i oczekiwanego kontekstu.

## Błędy i timeouty

- Zasymulować timeout providera przed i po wykonaniu zewnętrznej operacji.
- Wykonać retry po częściowym błędzie.
- Potwierdzić, że system nie fail open.
- Potwierdzić, że recovery nie duplikuje payments, refunds, bookings ani notifications.
- Sprawdzić, że nieznane stany wymagają bezpiecznej reconciliation, a nie automatycznego sukcesu.

## Testy wynikające z ukończonych labów

### Cena kontrolowana przez klienta

- Przesłać `price=0001` dla produktu o wysokiej wartości.
- Backend musi użyć ceny serwerowej albo odrzucić request.
- Finalny checkout musi ponownie obliczyć autorytatywną sumę.

### Workflow kuponów

- Zastosować `NEWCUST5`, potem `SIGNUP30`, a następnie ponownie `NEWCUST5`.
- Drugie użycie kuponu jednorazowego musi zostać odrzucone w pełnym zakresie koszyka lub historii klienta zdefiniowanym przez promocję.
- Checkout musi niezależnie zweryfikować kompletny zestaw zastosowanych promocji.
