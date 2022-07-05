* metody:
    * GET - pobieranie zasobu, bez jakiegokolwiek jego modyfikowania
    * POST - tworzenie nowego zasobu
    * PUT - aktualizacja zasobu w całości
    * PATCH - aktualizacja fragmentu/części zasobu
    * DELETE - usuwanie zasobu
* ścieżki (standardowe):
    * GET /zasób - fetch_many
    * GET /zasób/:id - fetch_one
    * POST /zasób - create
    * PUT /zasób/:id - update_whole
    * PATCH /zasób/:id - update_part
    * DELETE /zasób/:id - destroy
* ścieżki niestandardowe:
    * PATCH /zasób/:id/akcja - akcja zmieniająca zasób w części (np. eventy state machine)
    * PUT /zasób/:id/akcja - akcja zmieniająca zasób w sposób znaczący, lub nawet tworząca zasoby zależne (np. procesy wykonywane w tle, jak integracje
    * POST /zasób/akcja - akcja tworząca związane z zasobem inne zasoby (np. generowanie raportów, reindeksowanie bazy danych, itp.)
    * GET /zasób/:id/akcja - akcja zwracająca alternatywną reprezentację na podstawie zasobu (np. pre-rendering zależności dla formularza) 
* zagnieżdżanie ścieżek może zostać wprowadzone jedynie, gdy zagnieżdżony zasób faktycznie zależy od zasobu nadrzędnego i proces zarządzania wymaga obecności zasobu nadrzędnego
* statusy:
    * 200 ok - dla:
        * udanego fetch_one
        * udanego, całościowego fetch_many
        * udanego update_whole, nie zwracającego ETag
        * udanego update_partial, nie zwracającego ETag
        * udanego destroy zwracającego metadane
    * 201 created - dla:
        * udanego create
    * 202 accepted - dla:
        * udanego create procesowanego w tle
        * udanego update_whole procesowanego w tle
        * udanego update_partial procesowanego w tle
    * 204 no_content - dla:
        * udanego destroy
        * udanego update_whole zwracającego ETag
        * udanego update_partial zwracającego ETag
    * 206 partial_content - dla:
        * udanego, częściowego fetch_many
    * 304 not_modified - dla:
        * udanego fetch_one przy włączonym cache i braku zmian
        * udanego fetch_many przy włączonym cache i braku zmian
    * 400 bad_request - dla:
        * dowolnego endpoints, który nie może sparsować zapytania, LUB dla niespełnionej restrykcji strong_params
    * 401 unauthorized - dla:
        * dowolnego endpointa, gdzie wymagana AUTENTYKACJA nie jest spełniona
    * 403 forbidden - dla:
        *  dowolnego endpointa, gdzie wymagana AUTORYZACJA nie jest spełniona
    * 404 not_found - dla:
        * dowolnego endpointa, gdzie wymagany zasób nie został odnaleziony
    * 422 unprocessable_entity - dla:
        * dowolnego endpointa, gdzie walidacja payload nie przechodzi
    * 416 requested_range_not_satisfiable - dla:
        * częściowego fetch_many, gdzie walidacja Range nie przechodzi
    * 423 locked - dla:
        * dowolnego endpointa PUT/PATCH/DELETE, gdzie modyfikacja zasobu została wyłączona
    * 507 insufficient_storage - dla:
        * dowolnego błędu próby zapisu zasobu - problemy z połączeniem/zapisem w bazie danych, redisem, zapisu na dysku lub dowolnym innym serwisie…
* transakcje
    * transakcja zwraca nam zasób
    * efektem ubocznym transakcji jest jej przerwanie w konkretnym stepie
    * stepy powinny odpowiadać oczekiwanym statusom HTTP, ale przede wszystkim weryfikacji i procesowania requestu
    * standardowe stepy transakcji:
        * authenticate - autentykacja
            * na wejściu otrzymuje header Authentication
            * parsuje go
            * weryfikuje go
            * przekazuje dalej argumenty transakcji z DOKLEJONYM obiektem policy (użytkownik znaleziony przy pomocy nagłówka Authentication, policy class przypięte do transakcji)
            * fail 401
        * authorize - autoryzacja
            * fail 403
        * parse - parsowanie payloadu (w railsach prawdopodobnie ten step będzie nie potrzebny lub przeniesiony do controllera)
            * zmiana argumentów transakcji na obiekty zrozumiałe do dalszego procesowania
            * fail 400
        * validate - walidacja payloadu
            * przyjmuje payload
            * fail 422
        * parse_range - walidacja Range
            * przyjmuje header Range i tworzy z niego parametry dla query object
            * fail 416
        * find_*- poszukiwanie zasobu (collect w przypadku zbioru zasobów
            * przekazuje dalej argumenty transakcji z DOKLEJONYM obiektem znalezionego zasobu
            * fail 404
        * ensure_cache - weryfikacja warunku cache
            * weryfikuje ETag i warunki z nagłówków
            * „fail” 304
        * ensure_mutability - walidacja możliwości modyfikacji
            * fail 423
        * persist - zapis
            * fail 507
        * decorate - dekorowanie, prezentowanie, post processing…
    * wymagane „parametry” transakcji:
        * payload
            * w przypadku POST/PUT/PATCH - request body + query params + path params
        * range
            * w przypadku fetch_many - nagłówek Range
        * cache method
            * w przypadku GET - nagłówek Cache oraz If-*
        * authorization header
* rendering:
    * zawsze obiekt
    * zawsze conajmniej jedno z data oraz errors dostępne
    * data to obiekt:
        * dla pojedynczego obiektu
            * id - ID zasobu
            * type - kodowa nazwa/typ zasobu, rozpoznawalny przez klienta, wylistowany w dokumentacji
            * attributes - dane, które nas interesują
            * [opcjonalnie] relationships - relacje z innymi zasobami:
                * id - ID relatywnego zasobu
                * type - kodowa nazwa/typ relatywnego zasobu, rozpoznawalny przez klienta, wylistowany w dokumentacji
            * [opcjonalnie] meta - dodatkowe informacje
        * dla wielu obiektów to tablica w/w obiektów
    * errors to tablica:
        * code - kod błędu, najlepiej string, unikalny, najlepiej wylistowany w dokumentacji
        * [opcjonalnie] title - człowieko-czytacielny opis, I18n
        * [opcjonalnie] source - JSON pointer do błędu https://tools.ietf.org/html/rfc6901 w payloadzie
        * [opcjonalnie] meta - dodatkowe informacje
    * [opcjonalnie] meta - dodatkowe informacje
