System planowania działań

CZĘŚĆ I

Opis Problemu
1.	Jaki problem system rozwiązuje?
- System rozwiązuje problem zorganizowania czasu pracy oraz rozdysponowania poszczególnych zadań dla pracowników firmy, pozwala każdemu podejrzeć jakie aktualnie są zadania do zrobienia, sprawdzać ich postęp, czytać notatki czy kto aktualnie zajmuje się jakim zadaniem.

2.	Kto jest użytkownikiem?
- Programiści, menedżerowie, testerzy

3.	Skala systemu (liczba użytkowników / ilość danych)
- System jest ogólnoświatowy, można korzystać z tej samej tablicy będąc w Nowym Jorku tak jak i w Tokio. Liczba użytkowników może wynosić około 100.000 dziennie, czyli jakos 2.000.000 miesięcznie

Wymagania Biznesowe
1.	 Ogólny dashboard widoczny dla wszystkich, z możliwością dodawania zadań, aktualizowania ich, sprawdzania postępów, przypisywaniem osób, pisaniem notatek.
2.	Interfejs graficzny, umożliwiający rozróżnianie dla jakiej grupy osób są zadania, jakie mają priorytety. 
3.	Zapewnienie aktualizacji prac w czasie rzeczywistym dla wszystkich korzystających z danej tablicy, Wyświetlanie statusów kto nad czym pracuje w danym momencie, komunikaty o aktualizacjach w śledzonych zadaniach, podgląd innych zadań.
4.	Integracja z innymi aplikacjami wspierającymi prace zespołu, 
5.	Elastyczna konfiguracja, każdy użytkownik powinien mieć możliwość personalizacji według swoich upodobań, przestawiać kartki z zadaniami w inne miejsca niż domyślne, czy ustawiać inne kolory
6.	Możliwość generowania raportów, z poszczególnych zadań oraz poszczególnych pracowników do analizy pracy 
7.	Komunikator pozwalający rozmawiać członkom zespołu jak i między zespołami

CZĘŚĆ II

Opis komunikacji.
  Nadawca        Odbiorca        Mechanizm        Przesyłane dane

1. Komunikacja Synchroniczna
- Aplikacja Webowa - Api Gateway - HTTP/REST - Dane logowania, formularze zadań
- Api Gateway - Auth Service - RPC/Internal - Tokeny dostępowe. identyfikatory sesji, uprawnienia uzytkownikow
- Service layer - Auth Service - Internal - Żądania weryfikacji tożsamości
- Service layer - Redis - In Memory -  Szybkie zapytania o stan sesji, krótkotrwały cache danych
- Service layer - Database Cluster - SQL - Zapytania o treść zadań, dane pracowników
- Load Balancer - Web APP / Backend - HTTP - Ruch sieciowy uzytkownika rozdzielany na instancje serwerów

2. Komunikacja Asynchroniczna
- Service layer - Kafka - Event Bus - zdarzenia systemowe np. zadanie ukończone / nowy komentarz
- Kafka - Task Processing - Message queue - dane do ciezkich procesow, pliki do importu
- Kafka - Integration Service - Event / Webhook - Dane dla systemów trzecich
- Integration Service - Zewnętrzne integracje - Api / Webhook - Zdarzenia kalendarza, statusy repozytoriów
- Kafka - Notification Service - Pub/Sub - Id uzytkwnika docelowego, tresc komunikatu, typ powiadomienia
- Kafka - Raporty / Analiza - Batch Processing - Surowe dane o aktywności do raportów

3. Jakie dane są przesyłane
- SQL - Kto, co i kiedy zrobił. Kluczowe dla integralności systemu, muszą byc zapisywane synchronicznie aby uzytkownik mial pewnosc ze jego zmiana jest utrwalona
- Eventy - Któtkie komunikaty o zmianie statusu. Zamiast przesyłac całe zadania, system przesyła informacje np. "uzytkownik X zmienił status zadania Y"
- Websockety - Dane przesyłane do serwisow czasu rzeczywistego, które wypychają zmiany do przeglądarki użytkownika bez konieczności odświeżania strony

CZĘŚĆ III
1. Gdzie system jest rozproszony?
- Rozproszona warstwa danych - Zamiast trzymać dane uzytkownikow w jednym dysku, dane sa rozproszone na wiele serwerow
- Load Balancer - Zakłada, że pod spodem nei ma jednego serwera ale cała pula i rozdziela je zapobiegajac przeciazeniu maszyny
- Zdecentralizowane procesy w tle - Gdy użytkownik zleci wygenerowanie potężnego raportu, żądanie to nie obciąża głównego serwera. Zlecenie trafia do kolejki, a osobne, rozproszone serwery typu "Worker" podejmują to zadanie.

2. Jak system radzi sobie z częściową awarią?
- Awaria systemów przetwarzających w tle np Notification service
Gdy użytkownik coś zapisuje, service Layer dodaje to do bazy danych i wysyła zdarzenie do kolejki Kafka. Zdarzenia zaczynają sie buforować w kolejce, a gdy serwis wróci do życia automatycznie pobierze wszystko z kolejki zdarzeń Kafki i nadrobi zaległości.
- Awaria jednego z serwerów aplikacji
Load balancer stale sprawdza ich stan, jeśli zauważy, że instancja nr X przestaje odpowiadać, wykreśla ją z listy i przekierowuje cały ruch na pozostałe instancje, system działa troche wolniej ale dalej działa
- Awaria usług zewnętrznych
jeśli jakaś usługa padnie, Integration Service odbiera błąd. Zamiast pokazywać go użytkownikowi, serwis stosuje taktyke ponawiaj próbę coraz rzadziej. Spróbuje wyslac webhooka za 5 minut, a jeśli znowu sie nei uda to za 10 minut.
- Częściowa awaria bazy danych
Dzięki zastosowaniu Shardingu, awaria jednego, który obsługuje np Europe nie wyklucza z użycia tego w Stanach

3. Jak wygląda strategia skalowania?
- Skalowanie infrastruktury i kosztów
- - Projekt zakłada pełne wykorzystanie chmury obliczeniowej np AWS, Azure
  - Ponieważ aplikacja jest globalna, nie może stać tylk ow jednym miejscu np Europie, ponieważ użytkownicy np. w Tokio mieliby spore opóźnienie. System zostanie wdrozony w kilku regionach geograficznych jednocześnie
  - Serwery w chmurze będą automatycznie się uruchamiać i wyłączać w zależności od natężenia ruchu co pozwoloi na obniżenie kosztów utrzymania.
  - Ciężkie pliki jak grafiki, skrypty będa serwowane z najbliższego węzła dla danego użytkownika
  - Wprowadzenie wsparcia technicznego działającego 24/7 w różnych strefach czasowych aby reagować na awarie
  - zatrudnienie ludzi od Site Reliability aby zapewnić 99.9% dostępności systemu w skali roku
  - Dostosowanie systemu tak, aby dane obywatelii Unii Europejskiej były przechowywane jedynie w Europie zgodnie z RODO, i tak samo w innych krajach / kontynentach
  - Regularne zlecanie zewnętrznym firmom testów penetracyjnych, aby chronić dane firm korzystających z naszych tablic  

4. Jakie kompromisy wynikają z twierdzenia CAP?
- Spójność + odporność na podział
Jeśli serwery stracą łączność i nie mogą synchronizować danych, system zacznie odrzucać zapytania od użytkowników, aby mieć pewność, że nie zwróci starych nieprawidłowych danych.
stosowane w systemach bankowych, transakcjach finansowych czy rezerwacji biletów

- Dostępność + odporność na podział
Jeśli sieć padnie, każdy serwer nadal odpowiada na zapytania na podstawie danych jakie ma lokalnie. Przez jakiś czas może zwracać przestarzałe informacje, a kiedy sieć wróci serwer wyrówna braki
stosowane w Sieciach społecznościowych, systemach Iot, koszykach zakupowych

- Spójność + dostępność
Gwarantuje spójność i dostępność ale nie obsługuje awarii między węzłami. W praktyce oznacza to rezygnacje z rozproszenia i trzymanie całej bazy na jednym serwerze.

5. Jaki model spójności danych został przyjęty?
- Wykorzystany został model hybrydowy, kluczowe operacje jak modyfikacja statusu zadań czy zarządzanie uprawnieniami realizowane są w głównej rozproszonej bazie z zachowaniem silnej spójności, co gwarantuje integralnośc. Jednocześnie warstwy poboczne jak kolejkowanie zdarzeń, czy pamięc podręczna i powiadomienia oraz moduł do raportów opierają się na ostatecznej spójności.
