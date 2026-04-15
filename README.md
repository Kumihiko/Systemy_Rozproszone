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


