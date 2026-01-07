 # TIPS vs RT1 – porównanie rozszerzone (perspektywa banku)

 _**Dokument referencyjny:** architektura, finality, płynność, ryzyko, routing, operacje._

 ## 1. Kontekst: po co w ogóle są dwa systemy instant w EUR

 SEPA Instant Credit Transfer (SCT Inst) wymusił możliwość rozliczeń natychmiastowych 24/7.
 Europa zdecydowała się na dwa równoległe rozwiązania:

 - **TIPS** – rozwiązanie banku centralnego (ECB), maksymalna finalność.
 - **RT1** – rozwiązanie rynkowe (EBA Clearing), szybkie wdrożenie i elastyczność.

 Dla klienta końcowego oba systemy wyglądają identycznie. Dla banku – różnice są fundamentalne.

 ## 2. Definicje i pozycjonowanie

 > **TIPS (TARGET Instant Payment Settlement)**
 >
 > System instant prowadzony przez ECB. Settlement odbywa się bezpośrednio na rachunkach
 > banków w banku centralnym (central bank money).

 > **RT1**
 >
 > System instant prowadzony przez EBA Clearing. Działa w modelu clearingowym,
 > z prefundingiem i rozliczeniem końcowym w TARGET2.

 ## 3. Architektura rozrachunku

 | Element | TIPS | RT1 |
 |---|---:|---:|
 | Właściciel / operator | ECB | EBA Clearing |
 | Typ pieniądza | Central bank money | Model izby rozliczeniowej |
 | Settlement | Bezpośredni, finalny | Finalny, ale pośredni |
 | Zależność od TARGET2 | Ograniczona | Istotna (end-of-cycle) |

 ## 4. Finality i ryzyko prawne

 **Finality** oznacza prawnie nieodwracalny rozrachunek.

 - W **TIPS** finality następuje w momencie zapisu na rachunku w ECB.
 - W **RT1** finality jest gwarantowana przez reguły izby, a settlement końcowy domyka się w TARGET2.

 Z punktu widzenia regulatora i audytu, TIPS jest postrzegany jako najczystsza forma finality.

 ## 5. Płynność i prefunding (core koszt)

 | Obszar | TIPS | RT1 |
 |---|---:|---:|
 | Gdzie leży prefunding | ECB | EBA Clearing |
 | Dostępność środków | Zamrożone 24/7 | Zamrożone 24/7 |
 | Postrzeganie treasury | Jak RTGS buffer | Jak clearing buffer |

 Dla obu systemów głównym kosztem nie są opłaty transakcyjne, lecz koszt utrzymywanej płynności (FTP).

 ## 6. Operacje 24/7 i IT

 - Brak cut-off – system działa ciągle.
 - Brak kolejki – przy braku prefundingu następuje **reject**.
 - Wysokie wymagania SLA, monitoring i incident management.

 Z punktu widzenia IT różnice są mniejsze niż z punktu widzenia treasury.

 ## 7. Routing – decyzje praktyczne

 > **Preferuj TIPS, gdy:**

 - bank stawia na maksymalną finality i zgodność regulacyjną
 - istotna jest reputacja i relacja z ECB

 > **Preferuj RT1, gdy:**

 - bank jest silnie zintegrowany z EBA (STEP2 / EURO1)
 - liczy się szybkość wdrożenia i elastyczność operacyjna

 ## 8. Najczęstsze błędy interpretacyjne

 - „RT1 to to samo co TIPS” – nie, inny model prawny i rozrachunkowy.
 - „Instant = tanio” – koszt robi płynność i 24/7 ops.
 - „Prefunding to detal techniczny” – to kluczowa decyzja treasury.

 ## 9. Podsumowanie

 TIPS i RT1 realizują ten sam cel biznesowy (instant payments), ale robią to w dwóch różnych filozofiach:

 - **TIPS** – central bank first, maksymalna finalność.
 - **RT1** – rynkowy clearing, większa elastyczność.

 ---

 *TIPS vs RT1 • wersja rozszerzona • druk A4*
