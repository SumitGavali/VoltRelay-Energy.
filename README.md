# VoltRelay Energy: Network Performance Analysis

*Where growth is leaking revenue, and what to fix first*

## Executive Summary

Between January 2024 and June 2025, VoltRelay's network handled 3.81M swap attempts (3.59M completed) and earned ₹22.9 Cr. Volume grew 2.8× and revenue 3.0×. Unit economics improved too: margin per swap moved from –₹11.6 to +₹7.4, driven by an 8.3% price rise (₹60 to ₹65 from July 2024) and fixed station costs spreading over more swaps.

Service quality did not scale with the business. In normal months about 4% of attempts fail, but every April to June the rate jumps to 9.5–11.9%, and absolute failures in Q2 2025 (77,445) were nearly double those of Q2 2024 (39,820). Failures cost ₹1.42 Cr in lost revenue over 18 months, or ₹2.04 Cr annualised at the latest run-rate.

Three concentrated problems explain most of the avoidable loss:

- **Station hotspots:** 15 stations (10% of the network) cause 24.7% of failures, and all 15 run Gen1 chargers.
- **Battery quality:** three Kyron lots (KY-2407, 2408, 2409) hold 1,461 packs worth ₹4.95 Cr and sit about 19 SOH points below peers.
- **Rider churn:** riders who hit service failures early churn more, and battery condition is the strongest churn driver we could measure.

Fixing the 15 hotspots alone recovers about ₹29 L a year. Protecting new riders is worth up to ₹23.7 L a year, and repricing the loss-making ZipDrop contract addresses ₹36 L of extra discounts already given.

## Analytical Approach

**Scope.** The analysis covers 3.81M swap attempts across 150 stations, 6,500 batteries, hourly cabinet telemetry, support tickets, fleet contracts, and 17,444 riders scored for retention.

**Data cleaning.** Every step was logged so results can be audited:

- **Firmware timestamp bug:** events from firmware v3.2.0 (139,490 rows) were recorded 5h30m off, consistent with a UTC-to-IST offset. We corrected them first, because hour-of-day peak analysis is wrong without it.
- **Deduplication:** we screened for offline-batch retries within 120 seconds. After the other fixes this found 0 duplicate rows, so no records were lost.
- **City standardisation:** 21 inconsistent city spellings were mapped to 6 cities (585 rows).
- **Other fixes:** 62,031 events from 2 test stations removed; 9,684 SoC/SoH values above 100% capped as calibration drift; 15,287 impossible distance readings set to missing; 36,588 tickets reclassified where comments contradicted the agent label; partial telemetry kept as missing rather than imputed.

**Methods.** Monthly KPI trends, a Pareto analysis of station failures, lot-level battery scorecards against same-type peers, a retention model on first-three-swap experience (driver importance measured by AUC drop), and a difference-in-differences read of the peak-pricing pilot in Bengaluru and Pune from October 2024.

**Limits.** The findings are observational, so they show association rather than proof of cause. Margin relies on stated cost assumptions, and CSAT was excluded because it is missing-not-at-random.

## Key Insights

**1. Failures spike seasonally while the business scales.** Outside April to June, the failure rate is stable near 3.9% despite a near-tripling of volume. The summer peaks (11.9% in May 2024, 9.5% in May 2025) are dominated by "no charged battery" and riders abandoning the queue. The Q2 rate improved year on year (9.4% to 7.9%), but volume growth means more riders hit a failure. Three-wheelers fail at twice the rate of two-wheelers (10.0% vs 5.0%), and failures peak in the 19:00 to 22:00 evening window.

<img width="2375" height="767" alt="03_q1_indexed_trend" src="https://github.com/user-attachments/assets/381b2770-3d2a-446b-9639-398f09484aa9" />


*Figure 1. Volume and revenue (first three months = 100) nearly triple, while margin per swap rises and the failure rate spikes each April to June.*

**2. Failures are concentrated and hardware-linked.** The top 15 stations cause 24.7% of failures, and 38 of 150 stations account for half. Hardware, not site type, separates them: Gen1 chargers fail 6.8% of swaps versus 4.4% for Gen2 and Gen3, while connectivity quality (5.4–5.9%) and location type (4.7–5.8%) barely differ. Delhi, Jaipur and Hyderabad hold every hotspot. Newer expansion waves perform better than launch stations (4.5% vs 5.8%), which weakens the case for opening more sites before fixing existing ones.

<img width="2376" height="1475" alt="06_q2_failure_concentration" src="https://github.com/user-attachments/assets/198e86e0-de15-49d4-9b2a-7302a2d91ba0" />


*Figure 2. The 15 worst stations (red bars) cause 24.7% of failures; three-wheelers and the 19:00 to 22:00 window fail most.*

**3. Heat slows charging and creates stock-outs, mainly on Gen1.** Average charge time rises from 74 to 84 minutes between cool and hot conditions. On Gen1 it climbs from about 88 minutes to roughly 156 minutes once cabinet temperature passes 45°C, and in those conditions about 38% of station-hours have zero charged two-wheeler packs. When zero packs are on hand the no-battery rate is 7.2%, against 3.1% with more than 10 packs. Gen2 and Gen3 are barely affected.

<img width="2675" height="696" alt="09_q3_heat_chain" src="https://github.com/user-attachments/assets/fd91f75f-1c12-4e66-bc19-c27c5ca6032c" />


*Figure 3. Above 45°C cabinet temperature, Gen1 charge time and zero-stock hours climb sharply; no-battery failures are highest when no charged pack is on hand.*

**4. One supplier's lots are failing early.** Kyron lots KY-2407, 2408 and 2409 (1,461 packs, 22.5% of the fleet) average about 61% state of health against roughly 80% for peers. They degrade 9.6 points per 100 cycles versus 3.1 for the rest and deliver about 52 km of range versus 63–90 km for other lots. Other Kyron lots are healthy, so the fault is lot-specific.

<img width="2826" height="755" alt="12_q4_battery_health" src="https://github.com/user-attachments/assets/95a4210c-e55b-408f-aa5b-c4aff996c681" />


*Figure 4. Kyron lots KY-2407, 2408 and 2409 cluster near 61% state of health, about 19 points below same-type peers.*

**5. Early service failures and battery condition drive churn.** Retention is 87.6% for riders with no early failure, 83.3% with one, 73.7% with two and 55.6% with three (just 9 riders), a steep cliff after the second failure. Overall, riders with any failure in their first swaps retain 5.1 points worse, and 2,370 riders (13.6%) were exposed. In the driver model, battery condition and range is the primary factor, early service quality is secondary, and price, station quality and rider profile are negligible. The model is modest (AUC 0.58), so it ranks drivers rather than predicting individuals.

<img width="2976" height="755" alt="17_q6_retention_curves" src="https://github.com/user-attachments/assets/6c327bbc-1018-4980-b514-9e09578cf56e" />


*Figure 5. Riders with early failures retain worse. The three-failure bar rests on only 9 riders, so read it as directional.*

**6. Commercial levers are uneven.** The peak-pricing pilot cut peak-hour share by 3.1 points and added ₹2.2 per swap overall: +₹5.0 for independent riders, but –₹2.6 for partners whose surcharge is not billable. The largest partner, FeastFly (14.0% of revenue, 20.4% of contribution), earns ₹2.85 per swap versus ₹2.02 for the network, so it is healthy, but its 10.0% discount sits just under the 10.9% break-even. ZipDrop is the real drain: its discount was raised from 12% to 28% in November 2024 and its margin fell from –₹6.4 to –₹8.4 per swap.

<img width="2975" height="801" alt="16_q5_partner_economics" src="https://github.com/user-attachments/assets/9a70857f-e4b0-44d4-822b-1076e8279122" />


*Figure 6. ZipDrop, HaulKing, CargoTuk and ParcelNest lose money per swap; FeastFly sits above the network average (dashed line).*

## Actionable Recommendations & Business Impact

| Priority | Action | Business impact |
|---|---|---|
| 1 | Fix the 15 hotspot stations (list below): upgrade or add cooling to Gen1 chargers before April | ≈ ₹29.0 L/yr recovered (15 stations carry 23.7% of the ₹2.04 Cr annual loss; 60% recovery assumed) |
| 2 | Quarantine lots KY-2407/2408/2409, stop deploying them to high-use stations, and file a supplier claim | ₹4.95 Cr of battery capital at stake; 1,377 packs (₹4.40 Cr) are already below end-of-life SOH |
| 3 | Protect each new rider's first three swaps with healthy, fully charged packs, plus outreach after two failures | Up to ₹23.7 L/yr of early-failure churn cost (about 121 excess riders lost) |
| 4 | Reprice ZipDrop toward the ~10–11% break-even discount | Addresses ₹36.4 L of extra discounts since the amendment |
| 5 | Renegotiate FeastFly before granting exclusivity; keep the discount at or below 10.9% | Protects a partner earning 20.4% of contribution |
| 6 | Extend peak pricing only to independents and billable partners | ≈ +₹5.0 per swap for independents, avoiding –₹2.6 on non-billable partners |

Recurring upside from priorities 1 and 3 is about ₹53 L a year, with ₹36 L from ZipDrop repricing and ₹4.95 Cr of battery capital to defend. These are separate pools and should not be treated as one total.

### Fix-first station list

Ranked by lost revenue. All 15 run Gen1 chargers and fail 6.8–8.5% of swaps, against a network average of 5.6%.

| Rank | Station | City | Failure rate |
|---|---|---|---|
| 1 | STN-JAI-138 | Jaipur | 8.2% |
| 2 | STN-DEL-043 | Delhi | 8.2% |
| 3 | STN-HYD-072 | Hyderabad | 7.7% |
| 4 | STN-DEL-050 | Delhi | 8.2% |
| 5 | STN-JAI-137 | Jaipur | 8.3% |
| 6 | STN-JAI-141 | Jaipur | 8.3% |
| 7 | STN-HYD-070 | Hyderabad | 7.2% |
| 8 | STN-JAI-140 | Jaipur | 8.1% |
| 9 | STN-DEL-040 | Delhi | 8.2% |
| 10 | STN-DEL-047 | Delhi | 8.0% |
| 11 | STN-JAI-134 | Jaipur | 8.0% |
| 12 | STN-HYD-065 | Hyderabad | 6.8% |
| 13 | STN-DEL-048 | Delhi | 8.5% |
| 14 | STN-JAI-142 | Jaipur | 8.3% |
| 15 | STN-HYD-077 | Hyderabad | 7.1% |

### Response to the budget proposals

- **More stations:** not as proposed. Fix the hotspots first, since new waves do not outperform a repaired network.
- **More batteries:** partial. Quarantine the bad lots and fix charge turnaround, then expand.
- **Network-wide pricing:** segmented. Extend only where the measured effect is positive.
- **Exclusive partner deal:** only with repricing.

**Next 90 days:** start with the hotspot fixes and the lot quarantine, since both are independent of seasonality and the next heat peak begins in April.
