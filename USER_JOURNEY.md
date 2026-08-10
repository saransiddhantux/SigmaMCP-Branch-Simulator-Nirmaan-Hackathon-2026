# User Journeys

## Persona

**Vikram, Zonal Head (Western Region).** Travels between branches; historically only learns a branch is underperforming at quarterly review, or after a competitor has already moved in.

## The journey, as built today

**1. Orientation.** Vikram opens the app. The map defaults to a national, zoomed-out view — five colored **zone circles**, each labeled with its average health score and branch count. The right panel already shows the Western Region's aggregate health and a leaderboard of its best/worst branches, since it's the zone nearest the map's default center.

**2. Drill into the zone.** Vikram clicks the Western circle (or scrolls to zoom). The map animates in; once the zoom crosses a threshold, zone circles give way to **cluster blocks** — rounded squares, one per geographic cluster of nearby branches (e.g., "Pune Cluster," "Surat Cluster"), each colored by average health. The right panel updates automatically to whichever cluster is now nearest the map's center — no extra click required.

**3. Drill into a branch.** Zooming further swaps cluster blocks for individual **branch pins**. Vikram's own branch — Pune, Hadapsar — sits inside an **alert banner** at the top: *"Branch health projected to slip 6 points next week."* Clicking "View reason" reveals the specific driver: two RM departures plus a competitor breaking ground 1.2km inside the catchment.

**4. Read the health picture.** The right panel now shows Pune's full detail: a 68/100 health gauge, four sub-scores, three KPI cards (RM contribution, disbursement mix, competitor density — each with a trend sparkline), and a micro-market insight: *"Retail demand up 18% in 30 days"* — a local "lipstick effect" signal, with supporting drivers available behind a "Why?" disclosure.

**5. Consider the options.** Below, four AI-ranked recommended actions are listed, top-ranked: *"Launch Saarthi Fleet Program pilot — +22% projected incremental business."*

**6. Enter the Strategy Studio.** Vikram taps "Simulate →". The view expands into the full-screen **Strategy Studio** (Decision Intelligence Workspace): Pune's live map fills the surface, and a workspace panel docks on the right. At the top, the AI's **hero recommendation** — "Expand MSME Working Capital, 3.0× ROI, 76% confidence, ₹10L investment, Low risk, 9-month payback" — answers "what's the smartest move here?" before he touches a single control.

**7. Explore and configure.** Below the hero, seven strategy cards let him compare interventions; a rollout configurator lets him set budget (₹5L–₹50L), timeline, deployment scope (Pilot → State), and target geography. Every change recomputes **nine live business KPIs** (incremental revenue, disbursals, customer growth, portfolio-health lift, NPA change, ROI, payback, cost, confidence) with animated counters — and the map reacts in real time: the coverage radius grows, affected branches glow, and a pulsing ROI badge sits on Pune. A 3-way **trade-off panel** flags the Best ROI / Fastest Payback / Lowest Risk / Highest Growth options, and an AI-reasoning accordion explains *why*.

**8. Decide & act.** The executive summary assembles the business case — justification, top risks, key assumptions, timeline. Vikram taps **Approve rollout** (or Save scenario / Compare later / Export business case). A confirmation toast fires, the action lands in a session-persistent "Recently actioned" list, and the studio closes back to the dashboard — where, as he pans, the right panel hands off to the aggregate leaderboard again. Same mental model, no mode switch to think about.


Leadership — Anjali, Head of Product Strategy (National)
This walks through the current implementation of the Leadership view — a separate single-file build (React 18 + Leaflet, no build step) that reuses the same Mudra 2.0 tokens as the Zonal Head view. Where Vikram's view is branch-operations first, this one is market-first.
Persona. Anjali, Head of Product Strategy (National). She owns the question "which product should we push into which city next?" — a launch/expansion call she assembles today by hand from scattered demand, competition, and existing-footprint reads. She does not manage day-to-day branch operations; she needs a national, comparative, per-product read, not one branch's health score.
The journey, as built today

**1. Land on the answer, not the map. Anjali opens the app to the Overview screen — the first of two views in a persistent left navigation rail (Overview, Map view). It leads with the recommendation itself: one card per loan product naming the best city to extend it into (Jaipur for Gold Loan, Pune for both LAP and Personal Loan in the current data), each with its score out of 100, band, and the top-3 ranked cities beneath it.

**2. See why now. Above the recommendations sits a "Market pulse" strip of moving numbers — gold price and its monthly change, the repo rate after recent cuts, retail credit growth, Tier-2 housing prices, industry gold-loan AUM growth, and rising unsecured-PL stress. Below them, a global & policy signals list ties each recommendation to its market logic: gold's rally lifting pledged-collateral values, cheaper wholesale funding widening NBFC spreads, LTV regulation favouring organised lenders, the Tier-2 housing upcycle strengthening LAP cover. Each product card carries a one-paragraph "why now" gist drawing on these signals, so the headline call is already defensible before she touches the map.

**3.Explore on the map. From any product card, "Explore on map" jumps to the second view with that product pre-selected — a live, pannable map of India with twenty candidate cities pinned at their real coordinates, from Ludhiana down to Madurai and Kochi. Each pin is coloured by its opportunity band, the single biggest opportunity pulses, and the right pane ranks all twenty cities with the top city spotlighted.

**4.Switch product. Using the top-right product selector she flips between Gold Loan, LAP, and Unsecured Personal Loan. Every pin recolours and the ranked list re-sorts instantly — the same twenty cities, scored independently per product, tell three different stories.

**5.Drill into a city. She clicks a pin (or a list row). The right pane swaps to that city's full breakdown: overall score, band, a one-line plain-English insight, the three underlying factors (demand / competitor density / existing footprint) each with a 0–100 value, a Strong/Moderate/Weak level and a one-sentence rationale, plus a recommended action and a confidence note. The clicked pin gets a selected outline and its tooltip stays open.

**6.Understand why. The insight sentence names whichever factor drove the score most — e.g. "underlying demand is the strongest signal at 79/100" — so the recommendation is defensible, not a black-box number.

**7. Decide. The output is a specific, defensible next move — launch product X in city Y first, pilot in Z, hold on the rest — the recommended action attached to each band, backed by the factor breakdown and the market signals that explain the timing, ready to take straight into a planning conversation.

What this enables
This is a market/product expansion-opportunity scanner that now opens with the recommendation rather than making Anjali assemble it: the Overview says where to extend each product and why the market supports it today, and the Map view lets her interrogate that call across all twenty cities. The scoring formula is real, transparent and deterministic; the per-city demand/competitor/footprint inputs and the market-pulse figures are hand-authored plausibility estimates, not live market data — stated plainly in the app itself.
