# PAYMENTS – SŁOWNIK + ROUTING (Payments Bible v1)
**Wersja docelowa (10–12 stron A4)** • perspektywa banku / treasury / IT payments

## 1. Fundamenty systemów płatniczych

### 1.1 RTGS (Real-Time Gross Settlement)
RTGS to system rozrachunku **brutto w czasie rzeczywistym**, w którym:
- każda płatność jest rozliczana osobno,
- w pełnej kwocie,
- bezpośrednio na rachunkach banków w banku centralnym.

RTGS = **ostateczny ruch pieniądza** (final settlement).

### 1.2 Payment vs Settlement
**Payment** to proces operacyjny: zlecenie → komunikaty → kontrole → routing → settlement.  
**Settlement** to fakt prawny: nieodwracalna zmiana sald w banku centralnym.

### 1.3 TARGET2 / T2
TARGET2 (T2) to **RTGS dla euro** prowadzony przez bank centralny strefy euro (ECB).  
Jeśli euro **nie przeszło przez T2**, to **nie było settlementu** (niezależnie od komunikatów/clearingu).

## 2. Clearing, netto, brutto

### 2.1 Clearing
Clearing to proces **liczenia zobowiązań**, a nie przesyłania pieniędzy:
- zbiera transakcje,
- liczy pozycje netto,
- przygotowuje instrukcję settlementu do RTGS.

### 2.2 Netto
Rozliczana jest **różnica zobowiązań**.  
+ niska potrzeba płynności  
– ryzyko kumuluje się do momentu settlementu

### 2.3 Brutto
Każda płatność osobno, pełna kwota.  
+ minimalne ryzyko  
– wysoki koszt płynności

### 2.4 Zasada uniwersalna
**Clearing liczy. RTGS rozlicza.**

## 3. Płynność, czas, ryzyko

### 3.1 Płynność (Liquidity)
Zdolność banku do zapłaty **tu i teraz**, a nie „w bilansie”.

### 3.2 Intraday liquidity
Saldo na początku dnia + spodziewane wpływy – zobowiązania wychodzące.  
Opóźnienia wpływów zwiększają bufory i zmieniają routing.

### 3.3 Liquidity event
Sytuacja, w której bank **nie może zapłacić na czas**, mimo że „ma pieniądze”.

### 3.4 Cut-off
Godzina graniczna. Po cut-off: settlement jutro, overnight cost, FX risk, reklamacje.

### 3.5 FX risk
Ryzyko kursowe przy opóźnionym settlement (zwłaszcza cross-currency).

### 3.6 FTP (Fund Transfer Pricing)
Wewnętrzna cena pieniądza w banku: koszt trzymania bufora płynności.

## 4. Routing – logika decyzyjna

Routing to **decyzja biznesowa**, nie techniczna.

### 4.1 Kolejność decyzji
1) Waluta  
2) Wymóg instant  
3) Deadline / cut-off  
4) Kwota i ryzyko  
5) Dostępna płynność (gdzie i kiedy)

### 4.2 Skutki braku płynności
- RTGS → kolejka
- Instant → reject
- Clearing → ryzyko przesunięte w czasie

## 5. Systemy płatnicze – przegląd globalny

### 5.1 Strefa EUR
- RTGS: TARGET2  
- Clearing: STEP2 / EURO1  
- Instant: TIPS / RT1  
Settlement zawsze kończy się w TARGET2.

5.2 Dania (DKK)
- **RTGS (platforma):** T2 (wspólna platforma Eurosystemu)
- **Waluta:** DKK
- **Final settlement:** rachunki w Danmarks Nationalbank
- **Status KRONOS2:** wycofany (decommissioned)
- **Instant payments:** krajowy clearing → settlement DKK w RTGS (platforma T2)

> Dania zrezygnowała z krajowej platformy RTGS (KRONOS2) na rzecz **wspólnej platformy T2**.
> Zmiana ma charakter **techniczny**, nie walutowy – pieniądz DKK pozostaje
> w pełni pod kontrolą Danmarks Nationalbank.

### 5.3 Szwecja (SEK)
- RTGS: RIX  
- Clearing: Bankgirot  
- Instant: Swish

### 5.4 Finlandia (EUR)
Jak strefa EUR: TARGET2, EBA clearing, TIPS / RT1.

### 5.5 Polska (PLN)
- RTGS: SORBNET2  
- Clearing: ELIXIR (KIR)  
- Instant: Express Elixir  
- Overlay: BLIK

### 5.6 Norwegia (NOK)
- RTGS: NBO  
- Clearing: NICS  
- Instant: NICS Instant

### 5.7 USA (USD)
- RTGS: Fedwire  
- Clearing: CHIPS  
- Instant: FedNow / RTP

## 6. EURO – architektura ECB / EBA

### 6.1 ECB
- prowadzi TARGET2,
- prowadzi TIPS,
- odpowiada za final settlement EUR.

### 6.2 EBA Clearing
- STEP2 (retail),
- EURO1 (high value),
- RT1 (instant).
Settlement EBA zawsze kończy się w TARGET2.

Zasada: **EBA liczy. ECB rozlicza.**

## 7. CLS – FX settlement (must-have)

### 7.1 Problem bez CLS
Bez CLS może się rozliczyć jedna noga FX, a druga nie → **Herstatt risk**.

### 7.2 CLS i PvP
CLS realizuje **PvP (payment-versus-payment)**:
- albo obie nogi FX są rozliczone,
- albo żadna.

### 7.3 Waluty CLS / non-CLS
Nie wszystkie waluty są w CLS. Non-CLS = wyższe bufory, wyższe ryzyko.

## 8. Cross-border / correspondent banking

### 8.1 Co to jest
Cross-border non-EUR opiera się na bankach pośredniczących i rachunkach nostro/vostro.

### 8.2 Gdzie „stoi pieniądz”
Najczęściej u korespondenta (compliance, cut-off, time zone).

### 8.3 EUR vs non-EUR
- EUR (SEPA): bardziej centralna infrastruktura
- non-EUR: więcej zależności bilateralnych

## 9. SWIFT – MT vs ISO 20022

### 9.1 SWIFT = komunikacja
SWIFT nie przesyła pieniędzy, tylko instrukcje.

### 9.2 MT (legacy)
Sztywne pola, mniej danych, więcej ręcznej interpretacji.

### 9.3 ISO 20022
Strukturalne dane, bogatszy kontekst, lepsze compliance i STP.  
ISO nie „przyspiesza settlementu” – zmniejsza błędy i ręczną obsługę.

## 10. Incident & crisis flows

### 10.1 IT incident vs business incident
- IT incident: system nie działa
- business incident: **pieniądz nie idzie**

### 10.2 Typowe kryzysy
RTGS down, clearing delay, SWIFT outage, liquidity shortfall.

### 10.3 Decyzje kryzysowe
hold / reroute / reject / manual settlement — IT dostarcza sygnały i narzędzia.

## 11. Słownik pojęć (referencja)
Nostro, Vostro, Prefunding, Batch, Queue, Finality, Rerouting, STP, PvP, FTP.

## 12. Podsumowanie
Payments to nie systemy. Payments to **płynność + czas + ryzyko**.
