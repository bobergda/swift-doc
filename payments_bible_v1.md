# PAYMENTS — SŁOWNIK + ROUTING (Payments Bible v1)
**Perspektywa:** bank / treasury / IT payments  
**Cel:** wspólny język do rozmów o płatnościach (biznes–treasury–IT–operacje), bez mieszania „komunikacji” z „pieniądzem”.

> Zasada przewodnia: **clearing liczy, RTGS rozlicza, a routing decyduje**.

## Spis treści
1. Fundamenty: payment vs settlement
2. RTGS, clearing, brutto i netto
3. Płynność, kolejki, cut-offy
4. Warstwa komunikacji: SWIFT i ISO 20022
5. EUR/SEPA: T2, EBA, TIPS/RT1
6. Routing: logika decyzyjna i przykłady
7. Cross-border i korespondenci (nostro/vostro)
8. FX settlement: CLS i ryzyka
9. Ryzyka i kontrole (operacyjne, compliance, fraud)
10. Incydenty i kryzysy: runbook „pieniądz nie idzie”
11. KPI i obserwowalność (monitoring)
12. Słownik pojęć

---

## 1. Fundamenty systemów płatniczych

### 1.1 Payment vs settlement (najczęstsze źródło nieporozumień)
**Payment** to proces operacyjny end-to-end: zlecenie → walidacje → komunikaty → kontrole → routing → obsługa wyjątków.  
**Settlement** to fakt prawny/rozrachunkowy: **nieodwracalna zmiana sald** (finality) na rachunkach rozrachunkowych (zwykle w banku centralnym).

Konsekwencje praktyczne:
- można „wysłać komunikat” i nadal nie mieć settlementu,
- można mieć „clearing” i nadal nie mieć settlementu,
- „czas płatności” w oczach klienta ≠ czas final settlementu w RTGS.

### 1.2 RTGS (Real-Time Gross Settlement)
RTGS to system rozrachunku **brutto w czasie rzeczywistym**, w którym:
- każda płatność rozlicza się osobno (gross),
- w pełnej kwocie,
- na rachunkach rozrachunkowych banków w banku centralnym,
- z finality zgodnie z regułami systemu (final settlement).

RTGS = **ostateczny ruch pieniądza** (tam „pieniądz zmienia właściciela” w sensie rozrachunku).

### 1.3 Clearing (co robi, a czego nie robi)
Clearing to proces **wyznaczania zobowiązań** między uczestnikami:
- zbiera transakcje (często w paczkach/batch),
- liczy pozycje netto (lub inne agregacje),
- przygotowuje instrukcje do settlementu w RTGS.

Clearing **nie jest**: „transportem pieniądza”, „gwarancją settlementu”, „finalnością”.

### 1.4 Brutto vs netto — trade-off
**Brutto (gross):**
- + minimalizuje ryzyko ekspozycji międzybankowej w czasie,
- − kosztuje płynność (każda płatność „zjada” intraday liquidity).

**Netto (net):**
- + oszczędza płynność,
- − kumuluje ryzyko do momentu settlementu (i przenosi je na zasady clearingu).

### 1.5 Reguła uniwersalna
**Clearing liczy. RTGS rozlicza.**  
Jeśli settlement nie przeszedł przez RTGS (lub inną finalną warstwę w danym schemacie), to „pieniądz” operacyjnie może „być w drodze”, ale rozrachunkowo go nie ma.

---

## 2. RTGS, clearing i „gdzie naprawdę siedzi pieniądz”

### 2.1 Rachunki i warstwy (mentalny model)
W większości krajów masz trzy warstwy:
1) **warstwa klienta** (księgi banku: rachunki klientów),  
2) **warstwa międzybankowa** (zobowiązania pomiędzy bankami),  
3) **warstwa rozrachunku** (rachunki rozrachunkowe w banku centralnym / final settlement).

Klucz: klient widzi „saldo”, ale bank musi dowieźć settlement na warstwie 3, żeby bilans i ryzyko się domknęły.

### 2.2 Kolejki (queueing) i priorytety w RTGS
RTGS zwykle ma mechanizmy kolejkowania:
- płatności oczekują na środki,
- priorytety (np. krytyczne, systemowe, klient high value),
- mechanizmy optymalizacji płynności (offsetting, gridlock resolution — zależnie od systemu).

W praktyce „RTGS działa”, ale settlement nie idzie, bo bank jest w kolejce z powodu płynności albo limitów.

### 2.3 Limity i kontrola ryzyka (przed wysłaniem)
Typowe kontrolki w banku przed wysłaniem:
- limity ekspozycji na korespondenta/uczestnika,
- limity produktowe (instant vs standard),
- limity AML/sanctions,
- limity operacyjne (np. maks. kwota dla kanału),
- „funds check” (czyli: czy mamy płynność tam, gdzie settlement będzie potrzebny).

---

## 3. Płynność, czas i koszty (to jest prawdziwa fizyka payments)

### 3.1 Płynność (liquidity) — definicja operacyjna
Płynność to zdolność banku do zapłaty **tu i teraz** na właściwym rachunku rozrachunkowym, a nie „w bilansie”.

### 3.2 Intraday liquidity — prosta równowaga
`saldo start dnia + wpływy oczekiwane − wypływy zobowiązań = bufor`

Jeżeli wpływy się opóźniają:
- rośnie koszt bufora,
- rośnie presja na routing do tańszych kanałów (netting),
- rośnie ryzyko przekroczeń cut-offów.

### 3.3 Liquidity event (częsty w praktyce)
Sytuacja, w której bank **nie może zapłacić na czas**, mimo że „ma pieniądze” w innym miejscu (inna waluta, inny rachunek, inna strefa czasowa, inny korespondent).

### 3.4 Cut-off — co realnie oznacza
**Cut-off** to godzina, po której:
- dana ścieżka (RTGS/clearing/korespondent) nie gwarantuje settlementu tego samego dnia,
- rośnie koszt overnight (płynność/FX),
- rośnie liczba reklamacji i zapytań,
- rośnie ryzyko, że płatność „wisi” na statusach (a klient chce odpowiedzi).

### 3.5 FTP (Fund Transfer Pricing) — „cena pieniądza” w routingu
FTP to wewnętrzna cena płynności:
- mierzy koszt utrzymywania buforów,
- dyscyplinuje routing (np. kiedy warto zapłacić instant, a kiedy lepiej netting),
- pozwala porównywać koszt kanałów (opłaty + koszt płynności + koszt ryzyka).

---

## 4. Warstwa komunikacji: SWIFT i ISO 20022 (czyli „co wysyłamy” vs „co się dzieje”)

### 4.1 SWIFT = komunikacja, nie pieniądz
SWIFT transportuje **instrukcje** i **statusy** między instytucjami.  
Pieniądz „rusza się” dopiero w warstwie settlementu (RTGS/rachunki rozrachunkowe).

### 4.2 MT (legacy) vs ISO 20022 (data-driven)
**MT**: sztywne pola, mniej danych, więcej interpretacji i ręcznych wyjątków.  
**ISO 20022**: strukturalne dane, bogatszy kontekst (np. adresy, identyfikatory), lepszy screening i wyższy STP.

ISO 20022 zwykle:
- nie „przyspiesza settlementu” sam z siebie,
- ale zmniejsza błędy, naprawy, ścieżki manualne i odrzuty (czyli realnie skraca czas end-to-end).

### 4.3 Przykładowy łańcuch wiadomości (bardzo uproszczony)
Zlecenie klienta (kanał banku) → komunikat do clearing/RTGS/korespondenta → statusy (accepted/rejected/pending) → settlement → potwierdzenia → wyciągi/raporty (recon).

W operacjach kluczowe pytania brzmią:
- **czy instrukcja została przyjęta?** (acceptance),
- **czy settlement został dokonany?** (finality),
- **gdzie jest blokada?** (compliance / płynność / cut-off / format / korespondent).

---

## 5. EUR/SEPA: architektura Eurosystemu i EBA (najczęstszy case w Europie)

### 5.1 T2 (TARGET) — RTGS dla EUR
T2 to warstwa **RTGS dla euro** (final settlement layer).  
Jeśli EUR nie przeszło przez T2 (albo inną finalną warstwę w danym schemacie), to settlement nie nastąpił — niezależnie od tego, ile komunikatów „poszło”.

### 5.2 EBA Clearing (clearing w EUR)
EBA dostarcza clearing (np. dla retail i high value — zależnie od usługi).  
Zasada robocza:
- **EBA liczy** (clearing),
- **Eurosystem rozlicza** (T2 jako settlement).

### 5.3 Instant w EUR: TIPS / RT1 (co jest „instant”, a co „final”)
Instant payments wymagają:
- schematu/rail (np. instant clearing/switch),
- natychmiastowych odpowiedzi (accept/reject),
- natychmiastowego settlementu lub mechanizmu zapewniającego finalność w trybie 24/7.

W praktyce routing instant to zawsze decyzja: koszt, płynność, dostępność, SLA, ryzyko odrzuceń.

### 5.4 SEPA (produktowo, nie technologicznie)
SEPA to zestaw reguł i schematów (np. przelewy, polecenia zapłaty) oraz standardów danych.  
To, że coś jest „SEPA”, nie znaczy, że zawsze idzie tą samą ścieżką operacyjnie (np. różne clearingi, różne okna).

---

## 6. Routing — logika decyzyjna (najważniejsza część dla banku)

Routing to **decyzja biznesowa z ograniczeniami technicznymi**, a nie odwrotnie.

### 6.1 Minimalny zestaw pytań routingowych (kolejność ma znaczenie)
1) **Waluta** i kraj/obszar (domestic / SEPA / cross-border)  
2) **Wymóg czasu**: instant / same-day / next-day  
3) **Deadline**: cut-offy kanałów, time zone  
4) **Kwota i typ**: retail vs high value, pilne vs standard  
5) **Ryzyko i kontrolki**: AML/sanctions, fraud, limity, regulacje  
6) **Płynność**: gdzie mamy środki (rachunek RTGS / konto korespondenta)  
7) **Koszt**: opłaty + koszt płynności (FTP) + koszt operacyjny (manual)  
8) **Fallback**: co robimy, gdy kanał nie działa albo rejectuje

### 6.2 Skutki braku płynności (jak to wygląda „na ziemi”)
- RTGS: płatność wchodzi do kolejki (czasem bez jasnej informacji dla klienta)  
- Instant: bardzo częsty efekt to **reject** (bo nie ma „czekania”)  
- Clearing/netting: ryzyko i settlement przesuwają się w czasie (czasem „maskując” problem do okna settlementu)

### 6.3 Prosty „decision tree” (szkic)
1) Czy klient wymaga instant?  
→ tak: wybierz rail instant, zrób natychmiastową walidację płynności i limitów; w razie braku — zdefiniowany fallback (np. standard) albo reject.  
→ nie: idź w standard (clearing/RTGS) w zależności od kwoty, cut-off, kosztu i ryzyka.

### 6.4 Przykłady routingowe (case studies)
**Case A: EUR, klient chce natychmiast, kwota mała**  
- wybór: instant rail (o ile uczestnictwo/availability)  
- ryzyka: większy odsetek odrzuceń przy brakach płynności, presja na 24/7 ops  

**Case B: EUR, kwota wysoka, „dziś” i ważna**  
- preferencja: RTGS/high-value (mniej ryzyka)  
- krytyczne: cut-off i płynność na rachunku rozrachunkowym  

**Case C: USD cross-border, klient chce „ASAP”**  
- realnie: zależy od korespondentów, stref czasowych i compliance  
- krytyczne: opłaty, potrącenia, screening, statusy i śledzenie

---

## 7. Cross-border i correspondent banking (nostro/vostro)

### 7.1 Definicje operacyjne
- **Nostro**: „nasz rachunek u was” (rachunek banku A w banku B)  
- **Vostro**: „wasz rachunek u nas” (rachunek banku B w banku A)  
- **Correspondent chain**: łańcuch pośredników, gdy nie ma bezpośredniego uczestnictwa w danym systemie.

### 7.2 Gdzie „stoi” pieniądz
W cross-border pieniądz najczęściej „stoi”:
- na rachunku nostro u korespondenta,
- w procesie compliance (wstrzymanie do wyjaśnienia),
- w oknach cut-off zależnych od stref czasowych.

### 7.3 Opłaty i „lifty” (ważne dla reklamacji)
W cross-border częste są:
- opłaty pośredników (lifting fees),
- różne modele opłat (kto ponosi koszt),
- potrącenia, które klient widzi jako „brakujące pieniądze”.

Routing powinien uwzględniać nie tylko „czy dojdzie”, ale też „czy dojdzie w pełnej kwocie” i „czy dojdzie przewidywalnie”.

---

## 8. FX settlement: CLS, PvP i ryzyko Herstatt

### 8.1 Problem bez CLS (Herstatt risk)
Bez mechanizmu PvP może się zdarzyć, że jedna noga FX zostanie rozliczona, a druga nie. To jest klasyczne ryzyko settlementowe.

### 8.2 CLS i PvP (payment-versus-payment)
CLS realizuje PvP:
- albo rozliczają się **obie nogi**,
- albo **żadna** (w sensie final settlementu).

### 8.3 Waluty CLS vs non-CLS
Nie wszystkie waluty są w CLS. Dla non-CLS:
- większe bufory płynności,
- większy nacisk na limity i okna czasowe,
- więcej scenariuszy manualnych (zwłaszcza przy incidentach).

---

## 9. Ryzyka i kontrole (co naprawdę psuje płatności)

### 9.1 Mapa ryzyk w payments
- **Liquidity risk**: brak środków „tu i teraz” na właściwej warstwie  
- **Settlement risk**: opóźnienia/niepowodzenia final settlementu  
- **Operational risk**: awarie, błędy danych, procesy manualne  
- **Compliance risk**: sanctions/AML, KYC, false positives/true hits  
- **Fraud risk**: APP fraud, mule accounts, takeover, social engineering

### 9.2 Kontrole „must-have” przed wysłaniem
- walidacja danych beneficjenta (format, IBAN/rachunek, BIC gdzie wymagane),
- screening sankcyjny i AML (z obsługą eskalacji),
- kontrola limitów i płynności,
- reguły antyfraud (risk scoring, velocity, device, behawioralne).

### 9.3 STP (Straight-Through Processing)
STP to procent płatności, które przechodzą bez manualu.  
Najczęstsze „zabójcy STP”:
- słabe dane (adresy, identyfikatory, niespójne nazwy),
- niestrukturalne pola,
- różnice w interpretacji standardu,
- screening generujący masę false positives.

---

## 10. Incydenty i kryzysy: „pieniądz nie idzie”

### 10.1 IT incident vs business incident
- **IT incident:** system/komponent nie działa  
- **Business incident:** **pieniądz nie idzie** (SLA, reputacja, ryzyko finansowe)

Business incident może wystąpić nawet gdy „wszystko zielone” w IT (np. brak płynności, cut-off, odrzuty compliance).

### 10.2 Typowe kryzysy (checklista rozpoznania)
- RTGS opóźnia / kolejki rosną,
- clearing delay / missed window,
- outage komunikacji (np. gateway, sieć, SWIFT),
- spike odrzuceń (format/ISO validation),
- spike holdów compliance,
- liquidity shortfall (intraday).

### 10.3 Minimalny runbook decyzyjny
1) **Triaged**: które kanały i które waluty?  
2) **Scope**: ilu klientów / jaka kwota ekspozycji?  
3) **Action**: hold / reroute / reject / manual processing  
4) **Comms**: operacje, treasury, customer support, klienci kluczowi  
5) **Post-mortem**: root cause + poprawki w regułach routingu/monitoringu

---

## 11. KPI i obserwowalność (żeby routing nie był „czarną skrzynką”)

### 11.1 KPI operacyjne (praktyczne)
- end-to-end time (P50/P95) per kanał,
- udział instant vs standard,
- STP rate i przyczyny manualu,
- odsetek rejectów i top przyczyny,
- backlog kolejek RTGS i czas w kolejce,
- breach cut-off (ile i za ile),
- koszty: opłaty + FTP + koszty operacyjne.

### 11.2 Sygnały alarmowe (proste progi)
- nagły wzrost pending/queued,
- nagły wzrost rejectów w jednym kanale,
- „cisza” w przyjęciach/ack (brak statusów),
- divergence: dużo komunikatów wysłanych, mało settlementów potwierdzonych.

---

## 12. Słownik pojęć (referencja)
- **Acceptance** — przyjęcie instrukcji do przetwarzania (to nie settlement)
- **Batch** — przetwarzanie paczkowe (okna, cut-offy)
- **Clearing** — wyznaczanie zobowiązań
- **Cut-off** — godzina graniczna dla kanału/okna
- **Finality / final settlement** — nieodwracalny rozrachunek
- **FTP** — wewnętrzny koszt płynności
- **Gridlock** — zakleszczenie płynności (wszyscy czekają na wpływy)
- **Intraday liquidity** — płynność w trakcie dnia
- **Nostro / Vostro** — rachunki korespondencyjne
- **Prefunding** — prefinansowanie (typowe w instant/24x7)
- **PvP** — payment-versus-payment (FX)
- **Queue** — kolejka płatności oczekujących na środki/limity
- **Rerouting** — zmiana ścieżki po decyzji (np. fallback)
- **STP** — straight-through processing (bez manualu)

---

## Podsumowanie
Payments to nie „systemy i komunikaty”. Payments to **płynność + czas + ryzyko** — a routing jest mechanizmem, który te trzy rzeczy bilansuje.
