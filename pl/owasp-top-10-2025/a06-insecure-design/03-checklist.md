# A06: Insecure Design - Checklista

Używaj tej checklisty podczas projektowania funkcji, architecture review, planowania code review albo autoryzowanych testów.

## Business requirement i invariants

- Jaką regułę biznesową implementuje funkcja?
- Co musi być zawsze prawdą?
- Co nigdy nie może się wydarzyć?
- Jaki rezultat spowodowałby wpływ finansowy, prywatności, integralności albo dostępności?
- Czy reguła jest wystarczająco konkretna, aby ją przetestować?
- Czy regułę egzekwuje system, czy jest tylko opisana w UI?

## Assets i actors

- Jakie dane, tożsamość, funkcjonalność albo proces biznesowy są wartościowe?
- Kim są aktorzy?
- Którzy aktorzy są uwierzytelnieni, uprzywilejowani, zewnętrzni albo anonimowi?
- Który aktor tworzy największy impact po przejęciu konta?
- Czy role administracyjne są rozdzielone zgodnie z least privilege?

## Punkty wejścia i przepływy danych

- Gdzie dane lub komendy mogą wejść do systemu?
- Które wartości pochodzą z przeglądarki, aplikacji mobilnej, API clienta albo podmiotu trzeciego?
- Dokąd trafia każda wartość?
- Który komponent podejmuje finalną decyzję bezpieczeństwa?
- Czy pośredni serwis jest obdarzony zbyt dużym zaufaniem?
- Czy serwis docelowy samodzielnie weryfikuje operację?

## Trust boundaries

- Gdzie zmienia się zaufanie między klientem i serwerem?
- Gdzie zaufanie zmienia się między serwisami wewnętrznymi?
- Gdzie dane przechodzą do lub od third-party providera?
- Czy callbacki, webhooki i wiadomości są uwierzytelniane?
- Czy stan frontendu jest traktowany jako kontrolowany przez użytkownika?
- Czy direct API request może ominąć zamierzony UI flow?

## Frontend review

- Czy ukryty przycisk jest traktowany jako autoryzacja?
- Czy route guard jest jedyną ochroną uprzywilejowanej funkcji?
- Czy wartości z wyłączonych albo ukrytych pól można zmienić w żądaniu?
- Czy frontend wysyła autorytatywną cenę, rolę, ownership albo status?
- Czy API zwraca dane, które frontend jedynie odfiltrowuje?
- Czy kontrole client-side są zachowane dla UX bez mylenia ich z enforcementem?

## Uwierzytelnianie, role i sesje

- Czy tożsamość jest ustalana z poprawnej sesji lub podpisanego tokenu?
- Czy klient może przesłać lub zmienić swoją rolę?
- Czy każdy target service egzekwuje autoryzację?
- Czy tokeny resetu hasła są losowe, krótkotrwałe, jednorazowe i przypisane do konta?
- Czy odpowiedzi uwierzytelniania i odzyskiwania konta są odporne na enumeration?
- Które sesje są unieważniane po logout, password reset, password change, account disablement i role change?
- Czy wrażliwe zmiany roli wymagają recent authentication?

## Ownership i prywatność

- Czy ownership jest sprawdzany przy każdym read, update, delete i download?
- Czy reguły obejmują trasy alternate, batch i legacy?
- Czy prywatne rekordy są filtrowane przed serializacją odpowiedzi?
- Czy search, listing, detail, export i cache używają tych samych reguł dostępu?
- Czy pola odpowiedzi są allowlisted dla bieżącej roli i operacji?

## Workflow i stan

- Czy można pominąć krok?
- Czy można zmienić kolejność akcji?
- Czy tę samą akcję można powtórzyć?
- Czy token, kupon, webhook, płatność lub refund mogą zostać replayowane?
- Czy dozwolone przejścia stanu są modelowane jawnie?
- Czy bieżący autorytatywny stan jest sprawdzany przed każdą zmianą?
- Co dzieje się po cancellation, expiry, refund albo account disablement?
- Czy równoległe żądania mogą naruszyć pojemność, saldo lub unikalność?

## Płatności i dostawcy zewnętrzni

- Czy kwota jest obliczana z zaufanych danych po stronie serwera?
- Czy currency, booking, customer i refund są prawidłowo powiązane?
- Czy operacje płatności i refundu są idempotentne?
- Czy podpis webhooka jest weryfikowany na oczekiwanym raw payload?
- Czy timestamp, event ID, event type, amount i currency są sprawdzane?
- Czy replayowane i stare eventy providera są odrzucane?
- Czy external provider ma wyłącznie minimalny wymagany dostęp?

## Zachowanie przy błędach

- Czy system fail closed przy timeout zależności?
- Czy error path może ominąć autoryzację albo walidację stanu?
- Co dzieje się po częściowym błędzie między serwisami?
- Czy retry może zduplikować płatność, refund, booking albo notification?
- Czy nieznane stany są odrzucane zamiast automatycznie akceptowane?
- Czy recovery jest bezpieczne i idempotentne?

## Wymagania bezpieczeństwa

Dobre security requirement powinno być:

- konkretne,
- testowalne,
- połączone z assetem i realistycznym zagrożeniem,
- niezależne od frontendu,
- możliwie niezależne od szczegółu implementacji.

Przykład:

> Booking Service musi weryfikować ownership rezerwacji przy każdej operacji read, update, cancellation i ticket download.

## Weryfikacja

- Jaki negative test dowodzi, że kontrola działa?
- Jaki stan musi pozostać bez zmian po odrzuceniu?
- Jaka odpowiedź i audit evidence powinny powstać?
- Czy test obejmuje direct API access, a nie tylko UI?
- Czy test obejmuje ponowne użycie, zmianę kolejności lub współbieżność, gdy ma to znaczenie?
