# Flight Delay Prediction Dashboard

*⭐⭐⭐⭐ Built · see live dashboard*

> Exported from the portfolio page on 2026-10-19. The live page is the canonical version: https://bhawnamanglapm.github.io/travel-tech-more.html#case-flight-delay-prediction-dashboard

- 8,000 flights analyzed
- Self-contained HTML/JS dashboard
- CVD-validated palette
- Full dark-mode support

**Tools used:** Python (pandas, gamma-distribution modeling), HTML/CSS/JS.

### Why Nobody Acts on This Today

An OTA or airline can't act on delay risk it hasn't aggregated. A single flight-status page tells one traveller their flight is late; nothing rolls that up into the pattern a PM actually needs: which routes are structurally unreliable, which airlines run tighter operations, and when in the calendar or the day risk spikes. This project builds that rollup, answering the three questions a travel-tech PM would actually be asked before shipping something like the "delay-risk badge" recommended in the Flight Booking Teardown (T1): which routes delay most, how airlines differ, and how season and time of day shape delay risk.

### My Approach

The honest constraint first: this was built in an environment that can't reach DGCA, US BTS, or Kaggle to pull a real historical dataset. Rather than present scraped-looking numbers that weren't actually real, I built a modeled dataset instead and said so directly on the dashboard: 8,000 flights generated from parameters I chose and documented: fictional airline names (so no delay number is ever attributed to a real, identifiable carrier), real Indian city-pair routes (airport congestion at Delhi/Mumbai/ Bengaluru and effects like North Indian winter fog are public aviation-industry knowledge, not a claim about any company), and additive delay effects for route congestion, season, and time-of-day. Delay minutes are drawn from a **gamma distribution**, not a normal one: real delays are right-skewed with a hard floor at zero, and gamma is bounded at zero and long-tailed by construction, which a normal distribution isn't. The first pass came out at a 48.9% on-time rate (implausibly low for any real airline), so I re-tuned the distribution parameters until the fleet-wide numbers matched a plausible real-world range, while keeping the underlying patterns (winter fog, monsoon, evening cascade) intact. Full calculation, every parameter, and the reasoning behind each one are documented in the dashboard's own [Methodology section](https://bhawnamanglapm.github.io/projects/flight-delay-dashboard/dashboard.html#methodology), summarized here in full.
   Show more: full calculation methodology, parameter tables (every number, and why)

### Methodology: the calculation, step by step

For every one of the 8,000 simulated flights:

- Draw a **base delay** from Gamma(shape=1.3, scale=3.2): mean ≈ 4.2 minutes, right-skewed.
- Multiply by that flight's **airline reliability index** (a single fixed number per airline, see table below), the only airline-level effect in the model, kept deliberately simple so differences between the fictional carriers stay modest rather than dramatic.
- Multiply by a **time-of-day multiplier** (0.90 Early Morning → 1.55 Night) to model how delays cascade later into an operating day.
- Add a fixed **route congestion term** (see table below), scaled by 0.35, a stand-in for how much busier Delhi/Mumbai/Bengaluru are than smaller airports.
- If the route is monsoon-sensitive *and* the month falls in Jun–Sep, add an extra Gamma(1.6, 5.0) term (mean ≈ 8 min).
- If the route is fog-sensitive *and* the month is Dec/Jan, add an extra Gamma(1.8, 6.0) term (mean ≈ 10.8 min), amplified 1.3× for Early Morning departures, since fog is worst at dawn.
- Cancellation is decided separately, as its own probability: a 0.6% base rate, +1.8pp if monsoon-exposed that month, +2.4pp if fog-exposed that month. If cancelled, delay is zeroed; a cancellation is a different outcome from "a very late flight," not the same thing at the extreme end.
- A flight counts as **delayed** if its final delay exceeds 15 minutes (the standard on-time threshold) and it wasn't cancelled.

| Airline (fictional) | Type | Reliability index |
|---|---|---|
| BlueSky Air | Full-service | 0.75 (best) |
| IndiFly | LCC | 1.00 |
| Skyline National | Full-service | 1.15 |
| Metro Express | LCC | 1.25 |
| Coastal Wings | Regional/turboprop | 1.35 (worst) |

| Route | Congestion term (min) | Monsoon-sensitive | Fog-sensitive |
|---|---|---|---|
| DEL-BOM | 9 | — | Yes |
| DEL-BLR | 7 | — | Yes |
| DEL-GOI | 7 | Yes | Yes |
| DEL-HYD | 7 | — | Yes |
| DEL-CCU | 6 | — | Yes |
| DEL-AMD | 6 | — | Yes |
| DEL-PNQ | 6 | — | Yes |
| BOM-BLR | 6 | — | — |
| BOM-COK | 5 | Yes | — |
| COK-BOM | 5 | Yes | — |
| BLR-HYD | 4 | — | — |
| MAA-CCU | 4 | — | — |
| BLR-COK | 4 | Yes | — |
| HYD-MAA | 4 | — | — |

Why these specific numbers: not derived from any real source, chosen and then iterated. The first draft produced an implausible 48.9% on-time rate; I scaled the base gamma parameters and congestion multiplier down until the fleet-wide rate landed at 84.4%, inside a plausible real-world range, while preserving the relative shape of every effect (which season is worst, which route is worst, night vs. morning). What this deliberately doesn't model: real distributional parameters fit from a published dataset (none was reachable from this environment), or flight-to-flight correlation (a delayed inbound aircraft delaying its own next outbound leg). A real airline operations model would need that, and it's a materially harder problem than this exercise set out to solve.

### The Analysis

With the dataset built, I grouped it four ways (by route, by airline, by season, and by time of day), computing average delay, the share of flights delayed over 15 minutes, and cancellation rate for each. A single "worst route" or "worst season" number hides the risk that actually matters operationally, which is where two dimensions compound, so I cross-tabbed the top 8 routes against all 5 seasons into a heatmap instead of stopping at one-dimensional rankings.

### Key Findings

- **84.4% on-time** fleet-wide, 9.2min average delay, 1.1% cancellation rate.
- **DEL-GOI is the single worst route** (12.6min avg, 34% delayed >15min): the only route in the model that's both winter-fog- *and* monsoon-sensitive, so it takes a hit twice a year instead of once.
- **Winter is the worst season by a wide margin** (11.9min avg vs. 7.5min in Spring/Post-Monsoon): every Delhi-origin route in the heatmap spikes in December–January.
- **Delays compound through the day**: Night departures average 11.0min vs. 8.3min in the Early Morning, consistent with how real airline operations cascade delays leg-to-leg.
- **Airline-level spread is real but secondary**: the gap between the most and least reliable modeled carrier (7.6–10.7min) is smaller than the gap between Winter and Spring (7.5–11.9min). Route and season explain more than carrier choice does.

### Recommendations

What a PM would actually do with this, if it were live data instead of modeled data:

- **Surface delay risk during booking, not after.** Feed a route + season + time-of-day risk score into the delay-risk badge recommended in the Flight Booking Teardown, so a traveller sees it before they book, not after they're already delayed.
- **Pad schedule buffers seasonally, not year-round.** A flat on-time SLA misses that Delhi-origin routes need a materially wider buffer specifically in December–January, not every month.
- **Treat Night departures as an upstream ops problem.** Since delay compounds through the day in this model, padding the last flight of the day treats a symptom; the real fix is earlier-leg reliability.
- **Re-run this against real DGCA/BTS data before shipping anything downstream of it.** This dashboard's job was to prove the analysis and the product judgment, not to replace a real data pipeline.

### Where the Real Risk Sits

The chart itself is rarely a delay dashboard's real product risk. The risk is presenting modeled or incomplete data as if it were ground truth. The discipline that matters here is the same one from the AI Trip Planner's guardrails: say what's real, say what's modeled, and never let a polished visualization imply more certainty than the underlying data supports.
  [   [image: Flight Delay Prediction Dashboard: KPI tiles (8,000 flights, 84.4% on-time, 9.2min avg delay) and a bar chart ranking all 15 routes by average delay, DEL-GOI worst at 12.6 minutes]  ](https://bhawnamanglapm.github.io/projects/flight-delay-dashboard/dashboard.html)

Click the screenshot to open the live, interactive dashboard (also available in dark mode).

### Quick recall

| Section | Summary |
|---|---|
| **Problem** | A single flight-status page tells one traveller their flight is late, but nothing rolls that up into the pattern a PM actually needs — which routes are structurally unreliable, which airlines run tighter operations, and when risk spikes — the exact rollup needed before shipping a delay-risk badge like the one recommended in the MakeMyTrip teardown. |
| **Research steps** | Built 8,000 flights from documented parameters since no real DGCA/BTS/Kaggle dataset was reachable → chose a gamma distribution over a normal one because real delays are right-skewed with a floor at zero → the first pass came out an implausible 48.9% on-time, so re-tuned parameters until the fleet-wide numbers matched a plausible real-world range while keeping the underlying patterns intact → grouped the data 4 ways (route, airline, season, time of day) and cross-tabbed routes against seasons into a heatmap instead of stopping at one-dimensional rankings. |
| **Solution** | A self-contained, CVD-validated HTML/JS dashboard surfacing on-time rate, average delay, and cancellation rate by route/airline/season/time-of-day, with every modeling choice documented in its own Methodology section rather than presented as scraped real data. |
| **Conclusion** | DEL-GOI is the worst route specifically because it's the only one exposed to both winter fog and monsoon risk — a pattern invisible in a one-dimensional ranking. The real product risk was never the chart; it's presenting modeled data as if it were ground truth, which is why every claim on the dashboard is labeled as modeled, not measured. |

Link[live dashboard →](https://bhawnamanglapm.github.io/projects/flight-delay-dashboard/dashboard.html) Image[dark-mode screenshot →](https://bhawnamanglapm.github.io/projects/flight-delay-dashboard/screenshots/dashboard-dark.png)
