# Power 2026 — Electricity Pricing in the Age of AI

> **Source:** https://power2026.ai — a free online primer by **Neel Somani** (former quant researcher at a major hedge fund, covering power & gas; advisor to founders/investors on data-center buildouts), © 2026.
>
> **APA citation:** Somani, N. (2026). *Power 2026: Electricity pricing in the age of AI* [Online primer]. https://power2026.ai/
>
> **Captured:** 2026-07-31 · **Status:** knowledge capture + MC Studio bridge notes. Figures are the book's own unless marked *(analysis)*.
>
> ← [Wiki index](README.md) · Next: [Sita Sector Energy Playbook](sita-sector-energy-playbook.md)

---

## Why this book matters to MC Studio

The book's core claim: **the real constraint on AI (and on every digital economy) is power, not GPUs or memory.** Data centers already consume ~5% of US power, with demand doubling every ~2 years (Goldman Sachs projection: US data-center power demand doubles by 2027). The same dynamics are arriving in Kenya/East Africa: Microsoft + G42's $1B geothermal-powered data center, IXAfrica's 20 MW Nairobi expansion, and an East African pipeline of ~8 facilities under construction (2025) + ~14 more by 2030.

For the Sita Sector Program, this primer supplies the **mental model and financing playbook** for the community VPP / physical power plant thesis applied to each of the six priority sectors — see the [Sita Sector Energy Playbook](sita-sector-energy-playbook.md).

---

## Part 1 — All About Power Plants

### Ch. 2 · Fundamentals of Commodities Pricing

- **System balance equation** (must hold at every location): `Supply = Demand + Net Exports + Change in Storage`. For power: `Generation = Consumption + Net Exports + Change in Storage + Losses`.
- **Marginal pricing:** in a competitive market every unit sells at the *marginal cost* — the cost of the cheapest producer that must turn on to serve the last unit of demand. The marginal producer makes $0 profit on its last unit. This is why cheap producers (e.g., solar) capture outsized profits.
- **Why power is special:**
  - Injected/withdrawn at physical **buses**; transmission lines have **ratings** (heat → sag → fire risk), so flows are physically constrained.
  - **Storage** = batteries, pumped hydro, compressed air.
  - An **ISO/balancing authority** keeps frequency at 60 Hz; generators are $100M+ rotating machines. Brownouts/blackouts occur when supply can't match demand.

### Ch. 3 · The Power Plant

- **EIA-860** is the annual US census of every generator >1 MW (capacity, fuel, heat rate).
- **Heat rate** = MMBtu of fuel per MWh produced. Typical natural-gas unit ≈ **7 heat rate** (lower is better).
- **Gas plant architecture:** Simple-Cycle Gas Turbine (SCGT, ~300–400 MW) → add Heat Recovery Steam Generator → Combined-Cycle Gas Turbine (CCGT, ~600 MW). Inefficient **peakers** push output higher (~800 MW) at much worse heat rates.
- **Merit order:** generators ranked cheapest → most expensive by marginal cost; the ISO dispatches up the curve until demand is met. The last unit dispatched (often gas) **sets the price** — a uniform clearing-price auction pays *everyone* the same price.
- **Startup costs** (fuel burned while spinning up, wear, labor) make the offer curve upward sloping; operators dispatch most-efficient units first.
- **Revenue math example:** a 100 MW unit running 16h at $60/MWh ≈ **$100K/day gross**. Typical US prices: $10–150/MWh, sometimes higher.

### Ch. 4 · To Build A Power Plant

The development sequence: **Scoping → Site selection → Financing → Construction → Operation**.

1. **Scoping:** fuel type, size, and purpose decide site constraints (gas plant → near a pipeline). Grid export means joining the **interconnection queue** (long waits). Behind-the-meter (BTM) supply for a data center can avoid interconnection but needs space for GPUs + cooling + batteries.
2. **Site selection:** pick the plant type *then* the site. Deconflict with GIS files (oil & gas, mining claims, protected species — e.g., the Greater Sage Grouse Plan). Get permits from bodies like Washington's EFSEC; engage local counsel. Friendly states offer sales-tax exemptions for data centers (Virginia's Data Center Retail Sales and Use Tax Exemption).
3. **Financing:** a $300M+ plant is underwritten against **contracted cash flows**, not volatility:
   - **Long-term PPA** with an anchor tenant (hyperscaler/AI lab) — the longer, the better.
   - **Heat-rate call option (HRCO):** sell the plant's revenue stream to a hedge fund in exchange for fixed monthly payments — the underwriting-grade cash flow. Typical debt: ~**SOFR + 2.25%** for prime-tenant projects.
4. **Construction:** EPC contractors; turbine backlogs at GE Vernova, Siemens Energy, Mitsubishi Power are months-to-years; jet turbines are being repurposed; parts imported from China.
5. **Operation:** refinance at lower rates once built; operator may sell to an independent power producer (IPP).

### Ch. 5 · Case Study: Homer City

- Pennsylvania coal plant: 2 GW, three units, ~10 heat rate — uncompetitive vs 6–7 heat-rate gas. Shut down 2023 after ~50 years.
- **Redevelopment:** 4.4 GW natural-gas plant (~$10B) purpose-built for a data-center campus. EQT Corporation signed as gas partner; anchor tenants unannounced (financing/permit leverage). PA DEP air-quality permit approved Nov 2025; 1,000 workers on site.
- Takeaway: **existing site + permits + anchor tenant = the template for fast-tracked power development** — and the same template applies to Kenya's stranded/undercapitalized assets (e.g., idle industrial land, agri-residue sites).

### Ch. 6 · Meeting the Growing Demand

- **Policy mood:** hesitant/hostile — White House Ratepayer Protection Pledge (Mar 2026), the Power for the People Act (S.3682), some successful moratoriums (Monterey Park, CA was first city to ban data centers via ballot measure).
- **Not all data centers are bad:** in high-renewable regions they can *stabilize* prices (act like a battery by consuming at off-peak), reduce per-ratepayer fixed costs, and absorb excess wind that would otherwise drive prices negative (ERCOT).
- **Separate / inference-first grids:** "off-grid" data centers, and geographically distributed GPU clusters that follow the cheapest power hour-by-hour. Inference workloads don't need gigawatt colocation — they can travel to cheap power (the pattern edtech/fintech inference platforms can exploit).
- **Propensity to buy (real prices):**
  - SpaceX → Reflection: ~**$5,000/MWh** incl. ready GPUs, but with a 90-day out (hard to underwrite).
  - Anthropic → TeraWulf: **$19B lease, 400 MW over 20 years** from H2 2027 ≈ **$271/MWh** without GPUs (building + cooling included).
- **Author's recommendation:** temporarily tolerate inefficient/dirtier fuel to build "startup-speed" development capability; EPCs should co-own plants (align incentives); **transmission is the true bottleneck** in many regions.

---

## Part 2 — How To Trade Power

### Ch. 7 · Power Markets in the United States

| Market | Character |
|---|---|
| **PJM** | The OG, most liquid market (PA/NJ/MD + more, incl. Virginia — data-center capital of the US) |
| **MISO** | Coal retirements → renewables; large wind; IN/IL/MI popular for new data centers |
| **CAISO** | No coal; renewables + nuclear + gas; NP-15/SP-15 zones; duck curve; imports from PNW |
| **ERCOT** | Energy-only, isolated Texas grid; legendary $9,000/MWh spikes; negative prices from excess wind (generators paid to keep running by Production Tax Credits) |
| **SPP / NYISO / ISO-NE** | Wind-heavy; Manhattan demand sink vs cheap upstate nuclear that can't reach it; winter gas scarcity → oil |

- Traders trade **zones** (averages of buses), not individual nodes. **Retail price = wholesale + transmission/distribution + utility overhead** — fixed costs can be the majority of a bill. *(Analysis: the same split applies in Kenya — KPLC's retail tariff bundles generation + transmission + distribution + levies, which is why captive solar looks so attractive.)*

### Ch. 8 · Case Study: Alberta (energy-only markets)

- Alberta = "Texas of Canada": gas-heavy, energy-only market, AECO gas price.
- **The missing-money problem:** marginal-cost pricing can't pay for fixed costs of the least-efficient generators, so most US markets add a **capacity market / resource adequacy** ("payment for existing"). Texas/Alberta instead use **energy-only** design: scarcity prices (Texas adds VOLL ≈ $5,000/MWh × probability of lost load; Alberta caps at C$1,000/MWh).
- **Duck curve:** deep renewables penetration → daytime price ~$0, evening spike (SCGTs cheap to start). **Batteries flatten the curve** (charge cheap daytime, discharge evening) — the arbitrage that makes storage profitable.

### Ch. 9 · The Production Cost Model

- The ISO's optimizer: minimize total production cost subject to demand, transmission limits, generator constraints → outputs **Locational Marginal Prices (LMPs)**.
- Two spot markets: **day-ahead (DA)** clears the day before; **real-time (RT)** re-optimizes every **5 minutes**. Imbalances between scheduled and actual MWh settle at RT prices.
- Unit commitment is a hard **mixed-integer** problem (binary on/off, no-load costs, minimum runtimes, startup costs) — hence approximations (Ch. 10).
- **Two-node congestion example:** node A marginal cost $100/MWh, node B $10/MWh, 50 MW line. At 10 MWh of A demand, price = $10. At 60 MWh, the line saturates and A's price jumps to **$100** — a binding transmission constraint creates locational price divergence.

### Ch. 10 · Practical Approximations

Forecasting thousands of scenarios needs speed: (1) **continuous relaxation** (allow fractional on/off), (2) **exogenous interchange** (fix net flows instead of solving them), (3) **predictable battery dynamics** (charge day, discharge evening), (4) **merit-order dispatch** vs demand estimated from temperature, (5) **ignore/linearize losses**. Caveats: flows are the hardest part; startup costs and reliability constraints get lost. Still, "not so far off from what a trader can actually use."

### Ch. 11 · Types of Power Trades

- **Forwards:** lock a price for future delivery, settled financially against the DA price. Worked example: a 300 MW consumer buys a 1-year strip at $50/MWh; whether DA clears at $25 or $100, the hedge + spot purchase nets out to **$50/MWh**. Hedging a gas plant = sell power strip + buy gas strip (inherently short the spark spread).
- **Congestion trading / FTRs:** when a line binds, locational prices diverge; **basis trades** express a view on that divergence; **Financial Transmission Rights (FTRs)** pay off on congestion between two nodes.
- **Spreads:** the core vocabulary. **Spark spread = Power Price − (Heat Rate × Gas Price)** — the profitability of a gas plant. **Dark spread** = same with coal. **Effective heat rate** of a region = the implied heat rate at which a gas unit makes exactly $0 — used to predict which units run.
- **Outlook:** "incredible opportunity" for those who deeply understand power markets. Will new data centers raise or lower prices? Will inefficient sources get used? "Unsatisfyingly, it depends. But now you're equipped to answer."

---

## Key concepts glossary (one-liners)

| Term | Meaning |
|---|---|
| Merit order | Cost-ordered dispatch stack; cheapest units run first |
| Marginal unit / marginal pricing | The last unit needed sets the price everyone gets |
| Heat rate | MMBtu of fuel per MWh (lower = more efficient) |
| SCGT / CCGT | Simple-cycle / combined-cycle gas turbine |
| ISO / balancing authority | Nonprofit grid operator that clears the market & keeps frequency stable |
| LMP | Locational marginal price — price at a specific bus/zone |
| Day-ahead / real-time | Markets cleared day-before vs every 5 minutes |
| Capacity market / resource adequacy | Payments for generators to "exist" (insurance for reliability) |
| Energy-only market | No capacity payments; scarcity prices cover fixed costs |
| Duck curve | Daytime renewable glut + evening peak → storage arbitrage window |
| Spark/dark spread | Power price minus fuel cost at a given heat rate |
| FTR | Financial transmission right — payoff tied to congestion |
| PPA | Power purchase agreement — the anchor contract that enables financing |
| HRCO | Heat-rate call option — sells plant revenue for fixed payments |
| Behind-the-meter (BTM) | Generation consumed on-site, avoiding interconnection |
| VPP | Virtual power plant — aggregated DERs behaving like one plant |

---

## Bridge notes: applying this to Kenya *(analysis)*

- **Marginal pricing reality:** Kenya's grid is ~90% renewable (geothermal, hydro, wind, solar), but retail tariffs stay high (commercial ≈ KES 22–25/kWh; CI1 ≈ KES 13.44/kWh + ~KES 300/kVA demand) because costs are recovered through bundled charges — the book's "retail = wholesale + fixed costs" logic. This wedge is exactly what makes **captive solar + storage** profitable.
- **Net metering (<1 MW)** and **Open Access Regulations 2026 (>1 MW PPAs)** are Kenya's versions of "selling power"; they create the community VPP exit routes. **(Verify current status with EPRA — regulations are being revised 2024–2026.)**
- **Anchor-tenant financing** (Ch. 4) is the template for MC Studio's energy plays: secure the off-taker first, then build.
- **Duck curve dynamics** will arrive as Kenya's solar fleet (514 MW by June 2025 and growing fast) matures — early movers in storage/VPP aggregation capture the arbitrage.
- **Data-center-grade reliability** expectations (Microsoft/G42, IXAfrica) are raising the bar for all commercial/industrial power quality — power quality services become a sellable layer.

---

*Sources: power2026.ai (full text), EIA (Form 860, Today in Energy, spark-spread explainer), FERC ISO map, NERC, CAISO, AESO, PJM Manual 11, ISO-NE, William Hogan "Nodal Trading," Siemens Energy, Pillsbury Law, Homer City Redevelopment press releases, White House Ratepayer Protection Pledge (Mar 2026), Congress.gov S.3682, The Guardian (Monterey Park ban, Jun 2026), CNBC (bitcoin miners/ERCOT), DOE, Goldman Sachs. Full 41-footnote source list is on the site.*
