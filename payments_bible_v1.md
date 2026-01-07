# PAYMENTS - SLOWNIK + ROUTING (Payments Bible v1)
Perspektywa: bank / treasury / IT payments
Cel: wspolny jezyk do rozmow o platnosciach (biznes - treasury - IT - operacje), bez mylenia "komunikacji" z "pieniadzem".

Zasada przewodnia: clearing liczy, RTGS rozlicza, a routing decyduje.

## Spis tresci
1. Fundamenty: payment vs settlement
2. RTGS, clearing, brutto i netto
3. Plynnosc, kolejki, cut-offy, koszty
4. Warstwa komunikacji: SWIFT, ISO 20022, statusy
5. EUR/SEPA: T2, EBA, TIPS/RT1, produkty i R-transactions
6. Routing: logika decyzyjna, fallbacki, implementacja IT
7. Cross-border i korespondenci (nostro/vostro), tracking
8. FX settlement: CLS, PvP, non-CLS
9. Ryzyka i kontrole (operacyjne, compliance, fraud)
10. Incydenty i kryzysy: runbook "pieniadz nie idzie"
11. KPI i obserwowalnosc (monitoring)
12. Slownik pojec
Appendix A. Przeglad systemow (kraje)
Appendix B. Checklisty operacyjne

---

## 1. Fundamenty systemow platniczych

### 1.1 Payment vs settlement (najczestsze zrodlo nieporozumien)
Payment to proces operacyjny end-to-end: zlecenie -> walidacje -> komunikaty -> kontrole -> routing -> obsluga wyjatkow -> księgowanie.
Settlement to fakt prawny/rozrachunkowy: nieodwracalna zmiana sald (finality) na rachunkach rozrachunkowych (zwykle w banku centralnym).

Konsekwencje praktyczne:
- mozna "wyslac komunikat" i nadal nie miec settlementu,
- mozna miec "clearing" i nadal nie miec settlementu,
- czas platnosci w oczach klienta != czas final settlementu w RTGS.

### 1.2 Finality (dlaczego to slowo jest wazne)
Finality oznacza, ze po rozrachunku nie da sie "cofnac" transakcji standardowym mechanizmem technicznym.
Jezeli klient chce "anulowac przelew" po final settlement, to zwykle wchodza w gre:
- recall/request for return (prosba do beneficjenta / banku beneficjenta),
- proces reklamacyjny,
- postepowanie prawne.

Z punktu widzenia banku finality domyka:
- ryzyko settlementowe,
- ekspozycje miedzybankowe,
- zgodnosc ksiegowa i raportowa.

---

## 2. RTGS, clearing, brutto i netto

### 2.1 RTGS (Real-Time Gross Settlement)
RTGS to system rozrachunku brutto w czasie rzeczywistym, w ktorym:
- kazda platnosc rozlicza sie osobno (gross),
- w pelnej kwocie,
- na rachunkach rozrachunkowych bankow w banku centralnym,
- z finality zgodnie z regulaminem systemu.

RTGS = ostateczny ruch pieniadza (tam "pieniadz zmienia wlasciciela" w sensie rozrachunku).

### 2.2 Clearing (co robi, a czego nie robi)
Clearing to proces wyznaczania zobowiazan miedzy uczestnikami:
- zbiera transakcje (czesto w batchach),
- liczy pozycje netto (lub inne agregacje),
- przygotowuje instrukcje do settlementu w RTGS.

Clearing NIE jest:
- transportem pieniadza,
- gwarancja settlementu,
- finalnoscia.

### 2.3 Brutto vs netto - trade-off
Brutto (gross):
- + minimalizuje ryzyko ekspozycji w czasie,
- - kosztuje plynnosc (kazda platnosc "zjada" intraday liquidity).

Netto (net):
- + oszczedza plynnosc,
- - kumuluje ryzyko do momentu settlementu (i przenosi je na zasady clearingu).

### 2.4 Mentalny model warstw (gdzie "siedzi pieniadz")
W praktyce masz trzy warstwy:
1) warstwa klienta (ksiegi banku: rachunki klientow),
2) warstwa miedzybankowa (zobowiazania miedzy bankami),
3) warstwa rozrachunku (rachunki rozrachunkowe w banku centralnym / final settlement).

Klient widzi "saldo", ale bank musi dowiezc settlement na warstwie 3, zeby bilans i ryzyko sie domknely.

---

## 3. Plynnosc, kolejki, cut-offy, koszty

### 3.1 Plynnosc (liquidity) - definicja operacyjna
Plynnosc to zdolnosc banku do zaplaty "tu i teraz" na wlasciwym rachunku rozrachunkowym, a nie "w bilansie".

To rozroznienie jest krytyczne, bo bank moze miec nadwyzke:
- w innej walucie,
- na innym rachunku (np. u korespondenta),
- na rachunku klienta (ksiegi banku), ale nie na rachunku rozrachunkowym.

### 3.2 Intraday liquidity - prosta rownowaga
Saldo start dnia + wplywy oczekiwane - wyplaty zobowiazan = bufor.

Jezeli wplywy sie opozniaja:
- rosnie koszt bufora,
- rosnie presja na routing do tanszych kanalow (netting),
- rosnie ryzyko przekroczen cut-offow.

### 3.3 Kolejki w RTGS (queueing) i gridlock
W RTGS platnosci moga czekac w kolejce, gdy:
- brakuje srodkow na rachunku rozrachunkowym,
- dzialaja limity/priorytety,
- wystepuje gridlock (wszyscy czekaja na wplywy od innych).

Operacyjnie "system dziala", ale settlement "nie idzie", bo problem jest w plynnosci, a nie w sieci/IT.

### 3.4 Cut-off - co realnie oznacza
Cut-off to godzina, po ktorej:
- dana sciezka (RTGS/clearing/korespondent) nie gwarantuje settlementu tego samego dnia,
- rosnie koszt overnight (plynnosc/FX),
- rosnie liczba reklamacji i zapytan,
- czesc platnosci przechodzi na "jutro", nawet jezeli komunikat zostal wyslany "dzis".

### 3.5 Koszt platnosci to (oplaty + plynnosc + manual)
Przy porownywaniu raili nie porownuj tylko fee od systemu. Realny koszt to:
- oplaty zewnetrzne (rail, korespondenci, lifting),
- koszt plynnosci (bufory intraday/overnight),
- koszt operacyjny (manual, investigations, reklamacje).

### 3.6 FTP (Fund Transfer Pricing) - "cena pieniadza" w routingu
FTP to wewnetrzna cena plynnosci:
- pozwala przeliczyc "koszt trzymania bufora" na PLN/EUR/USD,
- dyscyplinuje decyzje routingowe (kiedy oplaca sie instant, a kiedy netting),
- pozwala porownac koszt kanalow w sposob mierzalny.

---

## 4. Warstwa komunikacji: SWIFT, ISO 20022, statusy

### 4.1 SWIFT = komunikacja, nie pieniadz
SWIFT transportuje instrukcje i statusy miedzy instytucjami.
Pieniadz "rusza sie" dopiero w warstwie settlementu (RTGS/rachunki rozrachunkowe).

Praktyczna konsekwencja:
- mozemy miec poprawnie wyslany komunikat, a mimo to brak settlementu (np. kolejka RTGS, hold compliance, cut-off).

### 4.2 MT (legacy) vs ISO 20022 (data-driven)
MT:
- sztywne pola, mniej danych,
- wiecej interpretacji i recznych wyjatkow,
- gorsze wsparcie dla compliance i recon.

ISO 20022:
- strukturalne dane, bogatszy kontekst,
- lepszy screening i wyzszy STP,
- mniejsza liczba bledow formatowych i "napraw" po drodze.

ISO 20022 zwykle nie "przyspiesza settlementu" sam z siebie, ale skraca czas end-to-end poprzez redukcje wyjatkow.

### 4.3 Lancuch wiadomosci (bardzo uproszczony)
Zlecenie klienta (kanal banku) -> walidacje -> routing -> wysylka do raila/korespondenta -> statusy -> settlement -> potwierdzenia -> raporty/wyciagi -> recon.

W operacjach kluczowe pytania:
- czy instrukcja zostala przyjeta (acceptance) i przez ktora warstwe?
- czy settlement jest finalny (finality)?
- gdzie jest blokada (compliance / plynnosc / cut-off / format / korespondent)?

### 4.4 ISO 20022 - mapa klas wiadomosci (sciagacz)
Nazwy roznia sie w detalach miedzy schematami, ale sens jest staly:
- inicjacja od klienta do banku: pain.* (np. polecenie przelewu, paczki)
- miedzybankowo (transfer): pacs.* (np. transfer srodkow)
- statusy: pacs.002 (accepted/rejected/pending - "co system zrobil z instrukcja")
- raportowanie/wyciagi/recon: camt.* (np. uznania/obciazenia, salda)

Najczestsza pulapka:
- accepted != settled

### 4.5 Odrzut, zwrot, recall - trzy rozne swiaty
W zaleznosci od raila i schematu spotkasz kilka klas zdarzen:
- reject: instrukcja nie weszla do przetwarzania (np. walidacja, brak plynnosci w instant)
- return: platnosc wraca po przetworzeniu (np. bledne dane beneficjenta)
- recall/cancellation: prosba o cofniecie/odwolanie (nie zawsze mozliwa; zalezy od finality i regul schematu)
- investigation/trace: zapytanie "gdzie jest platnosc" (czesto problem statusow i korespondentow)

---

## 5. EUR/SEPA: T2, EBA, TIPS/RT1, produkty i R-transactions

### 5.1 T2 (TARGET) - RTGS dla EUR
T2 to warstwa RTGS dla euro (final settlement layer).
Jesli EUR nie przeszlo przez T2 (albo inna finalna warstwe w danym schemacie), to settlement nie nastapil - niezaleznie od tego, ile komunikatow poszlo.

### 5.2 EBA (clearing w EUR) - zasada operacyjna
EBA dostarcza clearing (np. dla retail i high value - zalezne od uslugi).
Zasada robocza:
- EBA liczy (clearing),
- Eurosystem rozlicza (T2 jako settlement).

### 5.3 Instant w EUR: TIPS / RT1 (co jest "instant", a co "final")
Instant payments wymagaja:
- raila/schematu, ktory daje natychmiastowe odpowiedzi (accept/reject),
- mechanizmu zapewniajacego finality w trybie 24/7 (albo rozwiazania rownowaznego w danym ekosystemie),
- gotowosci operacyjnej (fraud/compliance/ops) w czasie zblizonym do 24/7.

### 5.4 SEPA - produktowo, nie technologicznie
SEPA to zestaw regul, schematow i standardow danych.
To, ze cos jest "SEPA", nie znaczy, ze zawsze idzie ta sama sciezka operacyjnie (rozne clearingi, rozne okna, rozne SLA).

### 5.5 Produkty SEPA: SCT, SCT Inst, SDD - co bank musi dowiezc
Z perspektywy banku "produkt" to nie tylko UI dla klienta, ale:
- reguly schematu (SLA, okna, limity, warunki zwrotow),
- reachability (do kogo realnie dojdziemy danym railem),
- obsluga wyjatkow i reklamacji,
- zasady ksiegowania i informowania klienta (kiedy pokazujemy "wykonano").

### 5.6 R-transactions (skrot: "R-ki")
Obsluga "R-ek" jest krytyczna, bo generuje:
- koszty operacyjne (manual),
- spory/reklamacje,
- ryzyko reputacyjne (klient: "wyslalem i nie doszlo").

Minimum, ktore powinno byc prawda w banku:
1) jednoznaczna klasyfikacja zdarzenia (reject/return/recall),
2) szybka informacja do kanalu klienta (status + powod w jezyku zrozumialym),
3) mozliwosc naprawy danych i ponownego wyslania (jesli ma sens),
4) recon i zamkniecie ksiegowe (zeby nic "nie wisialo").

---

## 6. Routing: logika decyzyjna, fallbacki, implementacja IT

Routing to decyzja biznesowa z ograniczeniami technicznymi, a nie odwrotnie.

### 6.1 Minimalny zestaw pytan routingowych (kolejnosc ma znaczenie)
1) waluta i obszar (domestic / SEPA / cross-border)
2) wymog czasu: instant / same-day / next-day
3) deadline: cut-offy kanalow, time zone
4) kwota i typ: retail vs high value, pilne vs standard
5) ryzyko i kontrolki: AML/sanctions, fraud, limity, regulacje
6) plynnosc: gdzie mamy srodki (rachunek RTGS / konto korespondenta)
7) koszt: oplaty + koszt plynnosci (FTP) + koszt operacyjny (manual)
8) fallback: co robimy, gdy kanal nie dziala albo rejectuje

### 6.2 Skutki braku plynnosci
- RTGS: platnosc trafia do kolejki (czasem bez jasnej informacji dla klienta)
- instant: czesty efekt to reject (bo nie ma "czekania")
- clearing/netting: ryzyko i settlement przesuwaja sie w czasie (czasem maskujac problem do okna settlementu)

### 6.3 Prosty decision tree (szkic)
1) Czy klient wymaga instant?
-> tak: wybierz rail instant, zrob natychmiastowa walidacje plynnosci i limitow; przy braku: zdefiniowany fallback (np. standard) albo reject.
-> nie: wybierz standard (clearing/RTGS) w zaleznosci od kwoty, cut-off, kosztu i ryzyka.

### 6.4 Case studies (po co routing istnieje)
Case A: EUR, klient chce natychmiast, kwota mala
- wybierz instant rail (o ile uczestnictwo/availability)
- ryzyka: wiecej odrzutow przy brakach plynnosci; presja na 24/7 ops; wieksze ryzyko fraud

Case B: EUR, kwota wysoka, "dzis" i wazna
- preferencja: RTGS/high-value (mniej ryzyka)
- krytyczne: cut-off i plynnosc na rachunku rozrachunkowym

Case C: USD cross-border, klient chce "ASAP"
- realnie: zalezy od korespondentow, stref czasowych i compliance
- krytyczne: oplaty/potracenia, screening, statusy, czas odpowiedzi na trace

### 6.5 Fallbacki i polityki reroutingu
Fallback to nie "awaryjny hack", tylko czesc produktu.
Powinienes miec jawne polityki:
- fallback po reject (np. z instant do standard),
- fallback po outage (np. zmiana clearingu/korespondenta),
- fallback po missed cut-off (przesuniecie na nastepny dzien lub inny rail),
- fallback po hold compliance (informowanie klienta + SLA).

Nie kazdy fallback jest dozwolony:
- nie zawsze mozna "zmienic rail" bez zmiany obietnicy produktowej (czas/koszt),
- nie zawsze mozna "ponowic" bez ryzyka duplikacji (idempotencja).

### 6.6 Routing jako produkt IT (jak budowac, zeby nie bolalo)
Najczestszy blad: routing zaszyty w if-ach w jednym serwisie. Minimum, ktore skaluje:
- reguly jako konfiguracja (wersjonowana, testowana, z audytem zmian),
- separacja danych wejsciowych od decyzji (normalizacja: waluta, kraj, typ, deadline, ryzyko),
- deterministyczna decyzja (ten sam input -> ten sam output) + idempotencja,
- explainability: "dlaczego poszlo ta sciezka" (powody w logach/raportach),
- fallback policy jako osobne reguly (co robimy przy niedostepnosci/odrzucie),
- feature flags i stop-the-line (mozliwosc szybkiego wylaczenia sciezki).

### 6.7 Testy routingu (praktyczne minimum)
Bez testow routing psuje sie "po cichu".
Minimum:
- testy regresji na bibliotekach przypadkow (kwota/waluta/cut-off),
- testy czasowe (cut-off, weekendy, swieta, zmiana czasu),
- testy niedostepnosci (symulacja outage -> oczekiwany fallback),
- testy compliance/fraud (hit/hold/reject -> oczekiwany flow i statusy).

---

## 7. Cross-border i korespondenci (nostro/vostro), tracking

### 7.1 Definicje operacyjne
- Nostro: "nasz rachunek u was" (rachunek banku A w banku B)
- Vostro: "wasz rachunek u nas" (rachunek banku B w banku A)
- Correspondent chain: lancuch posrednikow, gdy nie ma bezposredniego uczestnictwa w systemie docelowym.

### 7.2 Gdzie "stoi" pieniadz w cross-border
Najczesciej:
- na rachunku nostro u korespondenta,
- w procesie compliance (hold do wyjasnienia),
- w oknach cut-off zaleznych od stref czasowych.

### 7.3 Oplaty i "lifty" (zrodlo reklamacji)
W cross-border czeste sa:
- oplaty posrednikow (lifting fees),
- rozne modele oplat (kto ponosi koszt),
- potrącenia, ktore klient widzi jako "brakujace pieniadze".

Routing powinien uwzgledniac nie tylko "czy dojdzie", ale tez "czy dojdzie w pelnej kwocie" i "czy dojdzie przewidywalnie".

### 7.4 Tracking: "gdzie jest platnosc"
Cross-border generuje najwiecej pytan "gdzie jest przelew", bo:
- jest kilka instytucji po drodze,
- statusy sa asynchroniczne,
- compliance potrafi wstrzymac na poziomie posrednika.

Dobre praktyki:
- spójny identyfikator transakcji end-to-end (przechodzi przez warstwy),
- mapowanie statusow technicznych na statusy klienckie (bez wprowadzania w blad),
- SLA na investigations (ile czasu na odpowiedz, kiedy eskalacja do korespondenta),
- raportowanie "czas w poszczegolnych warstwach" (nie tylko czas total).

---

## 8. FX settlement: CLS, PvP, non-CLS

### 8.1 Problem bez CLS (Herstatt risk)
Bez mechanizmu PvP moze sie zdarzyc, ze jedna noga FX zostanie rozliczona, a druga nie. To klasyczne ryzyko settlementowe.

### 8.2 CLS i PvP (payment-versus-payment)
CLS realizuje PvP:
- albo rozliczaja sie obie nogi,
- albo zadna (w sensie final settlementu).

### 8.3 Waluty CLS vs non-CLS
Nie wszystkie waluty sa w CLS. Dla non-CLS:
- wieksze bufory plynnosci,
- wiekszy nacisk na limity i okna czasowe,
- wiecej scenariuszy manualnych (zwlaszcza przy incidentach).

### 8.4 Plynnosc FX w praktyce (dlaczego treasury jest w srodku)
Dla FX czesto dochodzi dodatkowa zlozonosc:
- prefunding po jednej stronie (zeby "miec z czego" rozliczyc noge),
- zaleznosc od korespondentow w walucie lokalnej,
- nieciaglosc okien settlementu (strefy czasowe + swieta lokalne).

---

## 9. Ryzyka i kontrole (operacyjne, compliance, fraud)

### 9.1 Mapa ryzyk w payments
- Liquidity risk: brak srodkow "tu i teraz" na wlasciwej warstwie
- Settlement risk: opoznienia/niepowodzenia final settlementu
- Operational risk: awarie, bledy danych, procesy manualne
- Compliance risk: sanctions/AML, KYC, false positives/true hits
- Fraud risk: APP fraud, mule accounts, account takeover, social engineering

### 9.2 Kontrole must-have przed wyslaniem
- walidacja danych beneficjenta (format, IBAN/rachunek, BIC gdzie wymagane),
- screening sankcyjny i AML (z obsluga eskalacji),
- kontrola limitow i plynnosci,
- reguly antyfraud (risk scoring, velocity, device, behawioralne).

### 9.3 STP (Straight-Through Processing)
STP to procent platnosci, ktore przechodza bez manualu.
Najczestsze "zabojcy STP":
- slabe dane (adresy, identyfikatory, niespojne nazwy),
- niestrukturalne pola,
- roznice w interpretacji standardu,
- screening generujacy mase false positives.

### 9.4 Compliance w payments (dlaczego wyglada jak "awaria")
Compliance/sanctions/AML generuje:
- hold bez przewidywalnego czasu zakonczenia,
- false positives (koszt i opoznienie),
- true hits (blokada i obowiazki raportowe).

Routing powinien to uwzgledniac:
- transakcje wysokiego ryzyka -> inny flow (np. dodatkowa autoryzacja),
- spodziewane holdy w cross-border -> uczciwe SLA do klienta.

### 9.5 Fraud i instant (dlaczego szybciej = trudniej)
Im szybsza platnosc, tym mniej czasu na detekcje.
Dla instant:
- wieksze znaczenie maja reguly pre-transaction,
- krytyczna jest mozliwosc natychmiastowego stop (jezeli rail na to pozwala),
- operacje musza miec szybkie sciezki eskalacji (war room).

---

## 10. Incydenty i kryzysy: runbook "pieniadz nie idzie"

### 10.1 IT incident vs business incident
- IT incident: system/komponent nie dziala
- business incident: pieniadz nie idzie (SLA, reputacja, ryzyko finansowe)

Business incident moze wystapic nawet gdy "wszystko zielone" w IT (np. brak plynnosci, cut-off, spike holdow compliance).

### 10.2 Typowe kryzysy (checklista rozpoznania)
- RTGS opoznia / kolejki rosną,
- clearing delay / missed window,
- outage komunikacji (gateway, siec, SWIFT),
- spike odrzutow (format/ISO validation),
- spike holdow compliance,
- liquidity shortfall (intraday).

### 10.3 Minimalny runbook decyzyjny
1) Triaged: ktore kanaly i ktore waluty?
2) Scope: ilu klientow / jaka kwota ekspozycji?
3) Action: hold / reroute / reject / manual processing
4) Comms: operacje, treasury, customer support, klienci kluczowi
5) Post-mortem: root cause + poprawki w regułach routingu/monitoringu

### 10.4 Role i decyzje (zeby nie bylo chaosu)
W kryzysie payments zwykle potrzebujesz jasnych rol:
- incident lead (jedna osoba od decyzji i priorytetow),
- treasury (plynnosc, limity, decyzje kosztowe),
- payments ops (kolejki, wyjątki, manual),
- IT (stabilizacja, workaroundy, komunikacja o ETA),
- compliance (jezeli jest spike holdow/true hits),
- customer comms (spojny przekaz: co wiemy i czego nie wiemy).

---

## 11. KPI i obserwowalnosc (zeby routing nie byl czarna skrzynka)

### 11.1 KPI operacyjne (praktyczne)
- end-to-end time (P50/P95) per kanal,
- udzial instant vs standard,
- STP rate i przyczyny manualu,
- odsetek rejectow i top powody,
- backlog kolejek RTGS i czas w kolejce,
- breach cut-off (ile i za ile),
- koszty: oplaty + FTP + koszty operacyjne.

### 11.2 Sygnały alarmowe (proste progi)
- nagly wzrost pending/queued,
- nagly wzrost rejectow w jednym kanale,
- "cisza" w przyjeciach/ack (brak statusow),
- divergence: duzo komunikatow wyslanych, malo settlementow potwierdzonych.

---

## 12. Slownik pojec (referencja)
- Acceptance - przyjecie instrukcji do przetwarzania (to nie settlement)
- Batch - przetwarzanie paczkowe (okna, cut-offy)
- Clearing - wyznaczanie zobowiazan
- Cut-off - godzina graniczna dla kanalu/okna
- Finality / final settlement - nieodwracalny rozrachunek
- FTP - wewnetrzny koszt plynnosci
- Gridlock - zakleszczenie plynnosci (wszyscy czekaja na wplywy)
- Intraday liquidity - plynnosc w trakcie dnia
- Nostro / Vostro - rachunki korespondencyjne
- Prefunding - prefinansowanie (typowe w instant/24x7)
- PvP - payment-versus-payment (FX)
- Queue - kolejka platnosci oczekujacych na srodki/limity
- Rerouting - zmiana sciezki po decyzji (np. fallback)
- STP - straight-through processing (bez manualu)

---

## Podsumowanie
Payments to nie "systemy i komunikaty". Payments to plynnosc + czas + ryzyko - a routing jest mechanizmem, ktory te trzy rzeczy bilansuje.

---

## Appendix A. Przeglad systemow (kraje) - skrot operacyjny
Uwaga: nazwy uslug/raili i ich zasady zmieniaja sie; traktuj to jako mape mentalna, nie dokument regulacyjny.

### A.1 Strefa EUR
- RTGS: T2
- Clearing: (rozne uslugi clearingowe w EUR)
- Instant: TIPS / RT1 (zalezne od uczestnictwa)

### A.2 Dania (DKK)
- Waluta: DKK
- Final settlement: Danmarks Nationalbank
- RTGS (platforma): T2 (wspolna platforma)
- Instant: krajowa infrastruktura + settlement w warstwie RTGS

### A.3 Szwecja (SEK)
- RTGS: RIX
- Clearing: Bankgirot
- Instant/overlay: Swish (dla uzytkownika)

### A.4 Finlandia (EUR)
- Jak strefa EUR: T2 + clearing w EUR + instant w zaleznosci od uczestnictwa.

### A.5 Polska (PLN)
- RTGS: SORBNET2
- Clearing: ELIXIR
- Instant: Express Elixir
- Overlay: BLIK

### A.6 Norwegia (NOK)
- RTGS: NBO
- Clearing: NICS
- Instant: NICS Instant

### A.7 USA (USD)
- RTGS: Fedwire
- Clearing (netting): CHIPS
- Instant: FedNow / RTP

---

## Appendix B. Checklisty operacyjne

### B.1 "Platnosc utknela" - 8 pytan, ktore oszczedzaja czas
1) Jaka waluta i jaki rail mial byc uzyty (standard/instant/cross-border)?
2) Czy instrukcja zostala przyjeta (acceptance) i na ktorej warstwie?
3) Czy mamy final settlement potwierdzony? Jesli nie - gdzie jest blokada?
4) Czy jest cut-off / okno settlementu, ktore minelo?
5) Czy jest w kolejce RTGS (liquidity) czy w holdzie compliance?
6) Czy to wyglada na blad danych (format, beneficjent, IBAN/BIC)?
7) Czy jest uruchomiony fallback/rerouting, czy czekamy?
8) Co i kiedy komunikujemy klientowi (status + SLA + next update)?

### B.2 "Spike rejectow" - szybka diagnostyka
- czy dotyczy jednego kanalu (np. instant) czy wszystkich?
- czy dotyczy jednej wersji formatu (walidacje ISO)?
- czy koreluje z godzina (cut-off/okno) albo zmiana konfiguracji?
- czy to problem plynnosci (instant/RTGS) czy walidacji danych (format)?

### B.3 "Duplikacja platnosci" - co sprawdzic zanim wyplacisz drugi raz
- idempotency key / identyfikator end-to-end: czy bank widzi to jako "ta sama" instrukcje?
- status settlement: czy pierwsza instrukcja jest finalnie settled, czy tylko accepted/pending?
- czy nie poszly dwa fallbacki (np. instant reject -> standard, ale klient ponowil i poszlo dwa razy)?
- czy recon/wyciagi potwierdzaja realne obciazenie rachunku rozrachunkowego?

