Presentation Notes
==================

ActionCable
-----------

ActionCable je novi dodatak za Rails 5 koji integriše WebSockets. To je full-stack rešenje, što znači da nudi JavaScript okvir na klijentskoj strani i Ruby okvir na serverskoj strani.

Usage
-----

ActionCable se koristi za aplikacije koje zahtevaju razmenu poruka u realnom vremenu. Primeri takvih aplikacija su:

 - softveri za online ćaskanje,
 - multiplayer video igre gde se npr. pozicija igrača na mapi šalje svim aktivnim igračima,
 - različite vrste izmena na stranici u realnom vremenu, npr. grafovi koji prikazuju podatke sa senzora
 - feeds na društvenim mrežama,
 - kolaborativno editovanje i kodiranje, npr. Google Docs i Cloud9 kao integrisano razvojno okruženje u browser-u

WebSocket
---------

ActionCable se nalazi na najvišem nivou apstrakcije što se radnog okvira tiče. To znači da su detalji protokola skriveni od korisnika. Za onoga ko koristi ActionCable korisno je da poznaje osnove WebSocket protokola.

WebSocket je TCP-bazirani nezavisni protokol. Jednom kada se uspostavi konekcija ona je trajna, za razliku od npr. HTTP protokola gde imamo otvaranje konekcije, slanje zahteva, čekanje na odgovor i zatvaranje konekcije. Kada je WebSocket konekcija uspostavljena, obe strane u komunikaciji mogu slati poruke u isto vreme. Protokol je dizajniran sa ciljem da bude implementiran u web browser-ima i web server-ima, ali se može koristiti i u drugim tipovima aplikacija. WebSocket podržava šifrovanu konekciju. WebSocket uvodi dve nove URI šeme: ws i wss.

WebSocket: Standards
--------------------

WebSocket protokol je standardizovao IETF(Internet Engineering Task Force) u dokumentu pod nazivom "The WebSocket Protocol", a WebSocket interfejs koji koristimo u JavaScript-u W3C(World Wide Web Consortium) u dokumentu "The WebSocket API".

U periodu pre i nakon standardizacije protokola koristila su se rešenja koja detektuju da browser ne podržava WebSockets, pa onda rade tzv. fallback na podržano rešenje za real-time komunikaciju poput: Polling, Long-polling, SSE itd.

WebSocket: can-i-use-it
-----------------------

S obzirom da je WebSocket protokol standardizovan pre 5 godina, svi proizvođači web browser-a su do sada implementirali ovaj protokol u svojim proizvodima. Google Chrome podržava WebSockets od 2010. godine, a Internet Explorer od verzije 10 tj. od 2011. godine.

Određena rešenja na tržištu poput Faye i Socket.IO detektuju da li klijent podržava WebSockets i u slučaju da klijent ne podržava WebSockets rade tzv. fallback na neko drugo rešenje za real-time komunikaciju: Polling, Long-polling, SSE itd. ActionCable podržava samo WebSockets.

WebSocket: Handshake
--------------------

WebSocket je nezavisan TCP-bazirani protokol koji je dizajniram sa ciljem da radi dobro sa postojećom Web infrastrukturom. Kao deo ovog dizajn principa, protokol specifikacija definiše da WebSocket konekcija počinje svoj život kao HTTP konekcija. Prebacivanje protokola sa HTTP na WebSocket se odvija u tzv. handshake fazi.

Termin handshake se u informacionim tehnologijama odnosi na automatizovani proces kojim se dinamički određuju parametri komunikacionog kanala koji se uspostavlja između dva entiteta pre normalnog toka komunikacije kroz taj kanal. Primer je SSL handshake gde se najpre vrši autentifikacija servera, a zatim kreiranje i razmena simetričnih ključeva.

WebSocket: Handshake Example
----------------------------

U handshake fazi razmenjuju se detalji konekcije. Browser inicira WebSocket konekciju i šalje zahtev serveru za nadgradnjom/zamenom protokola sa HTTP na WebSocket. Zahtev mora obavezno da sadrži HTTP verziju 1.1 ili veću i metod mora biti GET. Kroz zaglavlje "Upgrade: websocket" klijent zahteva nadgradnju/zamenu protokola. Ako server razume WebSocket protokol, šalje odgovor u kome se slaže sa zamenom protokola kroz "Upgrade: websocket" zaglavlje. Nakon toga, HTTP konekcija se zamenjuje WebSocket konekcijom.

WebSocket konekcija podrazumevano koristi iste portove kao i HTTP(80) i HTTPS(443).

S obzirom da se kroz HTTP zahtev šalju kolačići, moguće je utvrditi identitet korisnika koji inicira WebSocket konekciju i na osnovu toga autorizovati uspostavu konekcije. Ostala HTTP zaglavlja su dozvoljena, npr. Authorization za slanje JWT(JSON Web Token) radi Token-bazirane autentifikacije.

WebSocket: Visual Representation
--------------------------------

Nakon handshake faze i uspostave WebSocket konekcije omogućeno je slanje poruka u oba smera i u isto vreme. U svakom trenutku jedna od strana u komunikaciji može da zatvori konekciju.

WebSocket: API
--------------

WebSocket API je jednostavan.

Konekcija sa WebSocket serverom se uspostavlja kreiranjem WebSocket instance. Prvi argument se odnosi na URL do WebSocket servera.

Na instancu WebSocket konekcije mogu se dodati upravljači događajima(event handlers) koji se izvršavaju u trenutku otvaranja konekcije, primanja poruke i zatvaranja konekcije.

Slanje poruke ka serveru obavlja se funkcijom send.

Prekidanje konekcije sa klijentske strane obavlja se funkcijom close.

Terminology
-----------

Kao gotovo rešenje ActionCable sa sobom donosi niz novih termina.

 - **Connection** se odnosi na instancu klase tipa `ActionCable::Connection::Base` koju ActionCable kreira za svaku WebSocket konekciju.
 - **Consumer** ili potrošač je klijent WebSocket konekcije.
 - **Channel** predstavlja logičku jedinicu rada na koju se potrošač pretplaćuje. Kanal ima sličnu ulogu kao i kontroler u MVC-u.
 - **Subscriber** je klijent pretplaćen na jedan ili više kanala.
 - **Subscription** ili pretplata je veza između pretplatnika i kanala.
 - **Pub/Sub** skraćeno od Publish/Subscribe je messaging pattern gde pošiljalac poruke(publisher) ne šalje poruke direktno određenim primaocima poruka(subscribers), već poruke šalje na kanal bez znanja o tome ko će sve primiti poruku na tom kanalu.
 - **Broadcasting** se odnosi na slanje poruke svim pretplatnicima jednog kanala.

Pub/Sub Adapters
----------------------

ActionCable dolazi sa adapterima za Pub/Sub. Kao i drugi adapteri nudi jedinstveni interfejs bez obzira na izbor rešenja.

Podrazumevani adapter je async. Ovaj adapter je pogodan za korišćenje tokom razvoja aplikacije jer ne zahteva pokretanje dodatnih servisa.

Produkciono okruženju podrazumeva pokretanje Rails aplikacije u više instanci servera i procesa. Za svaku instancu servera i svaki proces(worker) koji server pokrene, kreira se nova ActionCable instanca. Upotreba async adaptera u produkcionom okruženju ne garantuje isporuku poruka ka svim konekcijama.

Redis je preporučeni subscription adapter koji koristi Redis Pub/Sub. Upotreba Redis adaptera za razliku od async garantuje isporuku poruka ka svim konekcijama.

PostgreSQL ima LISTEN/NOTIFY komande koje su slične PUBLISH/SUBSCRIBE kod Redis-a.

Running the ActionCable Server
------------------------------

ActionCable server može da se pokrene unutar aplikacije(In-App). To znači da se izvršava na istom portu kao i server aplikacije. Ovo je razlog zašto Rails 5 kao podrazumevani web server koristi Pumu umesto WEBrick. WEBrick ne podržava Rack socket hijacking API.

Kao Standalone ActionCable server je odvojen od servera aplikacije i izvršava se na drugom portu. Pokreće se kao nezavisna Rack aplikacija koja ima pristup svim domenskim modelima napisanim u ActiveRecord-u ili u drugim objektno-relacionim maperima.

Visualisation of ActionCable in Production
------------------------------------------

Upotrebom Redis-a i Redis adaptera moguće je pokrenuti veći broj ActionCable servera iza Load Balancer-a koji podržava WebSockets.

Code
----

Prilikom kreiranja nove Rails 5 aplikacije ActionCable kreira nekoliko novih fajlova. Najbitniji su, na serverskoj strani connection.rb, a na klijentskoj cable.js.

U fajlu connection.rb imamo definiciju klase `Connection` koja nasleđuje `ActionCable::Connection::Base`. U ovoj klasi se vrši autorizacija dolazeće konekcije. Ovo je deo handshake faze u WebSocket protokolu. Pretpostavlja se da je prethodno izvršena provera identiteta korisnika. Konekcije mogu da se identifikuju i na osnovu drugih vrednosti, ali je najčešći primer identifikovanje konekcije po korisniku. Kasnije se na osnovu korisnika mogu dohvatiti sve konekcije koje je taj korisnik otvorio i zatvoriti ih u slučaju brisanja korisnika, ili izmene prava pristupa.

U fajlu cable.js klijentska strana kreira konzumer instancu konekcije. Kao prvi argument funkcije `ActionCable.createConsumer()` navodi se adresa do ActionCable servera. U ovom slučaju ActionCable server se pokreće kao In-App pa nije potrebno navoditi adresu. Da bi kolačići bili dostupni serverskoj strani, neophodno je pokrenuti ActionCable server na istom domenu, ili koristiti drugačiji način provere identiteta korisnika, npr. Token-based authentication.

Naredni korak je generisanje kanala. Kao što je već rečeno, kanal je logička jedinica rada i ima sličnu ulogu kao i kontroler u MVC-u. Kreira se sa novim Rails generatorom za kanale, gde se navodi ime kanala i lista metoda koje će biti dostupne klijentskoj strani.

Generisanje kanala kreira dva fajla, od kojih je room_channel.rb namenjen serverskoj strani, a room.coffee klijentskoj strani.

U fajlu room_channel.rb na serverskoj strani imamo definiciju klase `RoomChannel` koja nasleđuje `ApplicationCable::Channel`. Metod `subscribed` se poziva u trenutku kada klijentska strana inicira pretplatu na ovaj kanal. Sa `stream_from` pozivom kreira se tzv. imenovani broadcasting koji se kasnije koristi za slanje poruka. Javni metod `speak` se zove direktno sa klijentske strane. U ovom konkretnom slučaju svi pretplatnici na ovaj kanal će dobiti istu poruku pri pozivu `ActionCable.server.broadcast` na prethodno imenovani broadcasting "room_channel". Moguće je suziti polje slanja poruke na specifičnu sobu tako što bi se "room_channel" argumentu dodao id konkretne sobe.

U fajlu room_channel.js na klijentskoj strani kreiranje pretplate se vrši pozivom `App.cable.subscriptions.create` sa imenom kanala na koji se klijent pretplaćuje. Ovaj poziv će na serverskoj strani pozvati metod `subscribed`. Funkcijom `perform` sve javne metode na kanalu na serverskoj strani, izuzev callback metoda, se mogu pozvati sa klijentske strane kao udaljeni pozivi (RPC). Kada server bradcast-uje poruku, metod `received` na klijentskoj strani će biti pozvan sa podacima sa servera. ActionCable vrši automatsku serijalizaciju/deserijalizaciju podataka koji se razmenjuju.
