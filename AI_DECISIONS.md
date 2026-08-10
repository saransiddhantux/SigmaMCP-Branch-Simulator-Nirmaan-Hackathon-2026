# AI / Simulation Logic Decisions

**Read this before demoing to anyone technical.** Everything in this document is a deterministic formula, not a trained model. That distinction is treated as a feature, not a limitation to hide — a judge who asks "is this real ML?" should get an honest "no, and here's exactly what it is instead" every time.

---

## 1. Branch scorecard generator (`buildAutoScorecard`)

**Inputs:** the branch's `locationCode` (used only as a seed) and `branchType` (Retail / Rural / Gold Loan).

**Reasoning:** a seeded pseudo-random number generator (`mulberry32`, seeded by hashing the locationCode) produces the same "random-looking" numbers for the same branch on every reload — stable for a demo, but genuinely random relative to any real branch performance.

**Outputs:**
- **Health score** (0–100): `50 + random(±18) + typeBias`, clamped to 28–95
- **4 sub-scores** (portfolio quality, micro-market demand, digital adoption, competitive resilience): each the health score perturbed by a further random spread
- **3 KPI cards** with plausible-looking values and trend sparklines (RM contribution ₹0.5–5Cr, disbursement mix 40–80% secured, 1–4 competitor rivals) — ranges chosen to look realistic, not derived from any real portfolio data
- **1 insight**, chosen from a fixed pool of 3 narrative templates
- **4 recommended actions** — as of 2026-08-09, ranked straight from `STRATEGY_CATALOG` (§3a below) via `rankedActionsForBranch()`, using the identical `strategyScore` formula that picks the Studio's own hero recommendation. This replaced an earlier, disconnected fixed `ACTION_POOL` whose titles didn't correspond to any Studio strategy at all — see `CHANGELOG.md`'s "Recommended Actions now open the Studio pre-selected..." entry for the full story. One consequence worth stating plainly: every non-hero branch's 4 actions are now the same 4 strategies scored the same way, so two branches with similar health/demand profiles will often show the same 4 titles — this is honest (it's the same deterministic formula, not a bug), not a sign of hidden per-branch variety that isn't actually there.
- **1 conditional alert**, auto-triggered whenever health < 48

**Business-logic assumption (explicit):** `typeBias` assumes Gold Loan branches skew healthier (+9, secured lending) and Rural branches skew lower (−8, typically higher risk/lower digital penetration in reality) — this is a modeling assumption chosen for narrative plausibility, not derived from actual Piramal portfolio data.

**Confidence:** none, in the statistical sense. There is no real confidence interval here — it's a fixed random spread designed to *look* like a distribution.

**Future ML possibility:** replace this entire function with a call to a real feature store + trained model (Layers 3–4 of `architecture_flowchart.png`) — the function's *output shape* (health, sub-scores, KPIs, insight, actions) was deliberately designed to be a drop-in-replaceable contract, so a real model could return the same shape without touching any rendering code.

---

## 2. Cluster derivation (k-means on lat/lon)

**Inputs:** every branch's real latitude/longitude, grouped by state.

**Reasoning:** k-means (8 iterations, deterministic evenly-spaced initial centroids — not random, so results are stable across reloads) with `k = round(branch_count / 7)`, targeting roughly 7 branches per cluster.

**Output:** a cluster ID and a display name (the most common city among that cluster's branches).

**Assumption (explicit):** this produces *geographically plausible* clusters, not Piramal's real organizational cluster boundaries, which weren't available in the source data. Documented in code comments and surfaced in `SETUP.md`'s Known Issues — never presented as authoritative.

**Future possibility:** replace with real cluster assignments the moment that data exists internally.

---

## 3. What-if simulation engine (`computeSim`)

**Inputs:** an action's baseline `[P10, P50, P90]` (itself synthetic, from #1 above), plus three user-chosen parameters: rollout intensity (1–100%), timeframe (3/6/12 months), intervention type (new product / staffing / promo).

**Reasoning:** three independent multiplicative factors, each expressed *relative to that action's own reference point* (70% intensity, 6 months, its own type):

- `intensityFactor(i) = 0.55 + (i/100) × 0.85` — 0.55× at minimum rollout, 1.4× at full catchment push
- `TIMEFRAME_FACTOR = {3mo: 0.55, 6mo: 1.0, 12mo: 1.6}`
- `TYPE_FACTOR = {NewProduct: 1.0, Staffing: 0.75, Promo: 0.6}`

The final scale is the product of each chosen value's factor divided by the reference value's factor — which is *why* the default slider positions always reproduce the original baseline number exactly, and why moving any single slider changes the result in an explainable, proportional way.

**Output:** a scaled `[P10, P50, P90]` range, displayed with a "Why this range" note that states the parameters used in plain language.

**Confidence framing:** presenting a P10/P50/P90 range (rather than one number) deliberately mimics how a real uncertainty-aware forecast would be communicated — Mudra's design principles favor this over false precision. **But it is important to be precise about what this is:** the range is a fixed-shape scaling of a random baseline, not a statistical confidence interval from any model. A judge asking "how was this range computed" should get exactly the explanation above, not a vaguer "the AI predicted it."

**Business-logic assumption (explicit):** the relative ordering of factors (new-product launches scale hardest with intensity/timeframe; promos scale least) reflects a plausible real-world pattern — new products need sustained rollout to show impact, promos are quick-hit — but is not calibrated against any real campaign data.

**Future ML possibility:** this is the clearest replacement target in the whole prototype. A real system would call a trained demand-forecasting model (per Layer 4 of `architecture_flowchart.png`) with the same three parameters as features, returning a genuine predicted-range output. The UI, parameter set, and "why this range" explanation pattern were all designed to survive that swap unchanged.

> **Note:** As of the Strategy Studio rebuild, this legacy `computeSim` engine is **dormant** (retained, not deleted). The primary what-if experience is now §3a below. `computeSim` is kept because its intensity/timeframe/type scaling philosophy informed the Studio engine and removing it risked breaking existing wiring.

---

## 3a. Strategy Studio economics engine (`computeStrategyProjection`) — the flagship what-if

This is the engine behind the Decision Intelligence Workspace. **It is a deterministic business-arithmetic model, not machine learning.** Same input → same output, every time.

**Inputs:** the selected branch, a strategy id (one of 7 in `STRATEGY_CATALOG`), and a live config `{budgetIdx, months, deployIdx, geoIdx}`.

**Reasoning:** each strategy carries hand-set economic coefficients per ₹1 lakh of budget (`revPerLakh`, `disbPerLakh`, `custPerLakh`, `healthDelta`, `npaBps`, `baseRisk`, `difficulty`, `baseConf`). The projection scales these by: budget (₹5L–₹50L), a deployment-reach multiplier (Pilot 1× → State ~30×) × geography reach, a timeline ramp (`{3:0.5, 6:1.0, 12:1.75}`), and a branch-demand tilt derived from the branch's own micro-market-demand sub-score. A per-(branch, strategy) seeded constant (`mulberry32(hash(branch+strategy))`, range 0.85–1.15) adds stable realism without breaking determinism.

**Outputs (the 9 live KPIs):** incremental revenue, incremental disbursals, expected customer growth, portfolio-health lift (pts), NPA change (bps), ROI (= revenue / cost — **internally consistent, verified in tests**), payback (= cost / monthly incremental margin, assuming ~22% margin realised over the horizon), implementation cost (= budget), AI confidence, plus a Low/Medium/High risk band and an affected-branch count.

**Hero pick & trade-off:** `recommendStrategy` scores every strategy by a composite `roi·2.2 + confidence·0.03 − riskPenalty − payback·0.04` and surfaces the winner as the hero card. `generateStrategyOptions` takes the top 3 and tags them Best ROI / Fastest Payback / Lowest Risk / Highest Growth for the comparison panel.

**Confidence framing:** the displayed "AI confidence %" is a **formula** (base confidence − difficulty − scope risk + data-richness + small seeded jitter), not a model-derived probability. A technical judge asking "where does 76% come from?" should get exactly that formula — never "the model is 76% sure."

**Business-logic assumptions (explicit):** coefficient magnitudes and their relative ordering (e.g. Collections Optimization improves NPA most; Digital Acquisition drives the most customers per lakh; Farmer Credit carries higher risk) are chosen for plausibility, **not** calibrated against real Piramal campaign outcomes. The 22% incremental-margin assumption is a placeholder.

**Future ML path:** replace `computeStrategyProjection` with a call to a real uplift/demand model (Layer 4) keyed on the same `(branch, strategy, config)` — the KPI bundle shape, the live-update contract, and the reasoning/summary panels were all designed to survive that swap unchanged.

---

## 3b. Top-2 action suggestions (`topActionSuggestions`) — the plain-English headline answer

> First applied to `~/Downloads/branch-pulse-view copy.html`; that copy has since been synced back as this folder's canonical file — see `CHANGELOG.md`'s 2026-08-09 entries.

**Inputs:** the selected branch and the live rollout config `{budgetIdx, months, deployIdx, geoIdx}`.

**Reasoning:** reuses `generateStrategyOptions(branch)` — the exact same fixed-baseline ranking that already powers the 3-way trade-off panel — and takes its top 2. Each is rendered through `STRATEGY_ACTION_TEMPLATES[strategy.id]`, a hand-written natural-language template per strategy (one of the 7 in `STRATEGY_CATALOG`) that substitutes in the branch's own projected numbers (customers, affected accounts) and the user's current rollout config (deployment scope, timeline). This is template substitution over an existing deterministic ranking, not free-text generation — no language model is involved anywhere in this path.

**Output:** exactly 2 sentences, e.g. "Increase RM capacity by 12 in the next 6 months." / "Create two different onboarding cadences for the digital acquisition push to capture demand quickly."

**Why top 2, and why re-derived rather than reusing whichever strategy is selected in §1:** the request was for "the top two things the user can do here" — branch-level advice — not a restatement of one manually-selected strategy. The ranking stays pinned to the same fixed baseline config used by the trade-off panel, so *which 2 strategies* surface remains a stable judgment about the branch; only the *phrasing* (headcount numbers, timeframe) reflects the user's live rollout settings.

**Business-logic assumption (explicit):** the numeric quantities inside each template (e.g. RM headcount as a function of deployment reach + budget tier) are hand-tuned to produce plausible, demo-appropriate magnitudes — not derived from any real staffing or capacity-planning model.

**Confidence:** none stated, deliberately. This surface is designed to read as a direct instruction ("do this"), not hedged with a percentage — the confidence/risk framing already lives one section down, in the KPI grid and reasoning accordion, for anyone who wants it.

**Future ML possibility:** the "branch → 2 ranked, phrased next-actions" contract was designed to survive a swap to a real recommender — `topActionSuggestions`'s output shape (2 plain sentences) would stay identical even if what selects and phrases them changes completely.

---

## 4. What is real, restated plainly

| Element | Real | Synthetic |
|---|---|---|
| Branch locations, names, types | ✅ | |
| Zone assignment (by state) | ✅ | |
| Cluster assignment | | ✅ (algorithmic, not official) |
| Health scores, sub-scores, KPIs | | ✅ |
| Insights (fixed template pool); recommended actions (ranked from `STRATEGY_CATALOG`, same formula as the Studio hero pick) | | ✅ |
| Legacy simulation output ranges (`computeSim`, dormant) | | ✅ (deterministic scaling, not a model) |
| **Strategy Studio projections** (revenue/ROI/payback/NPA/etc.) | | ✅ (deterministic economics formula, not ML) |
| **Strategy Studio "AI confidence %"** | | ✅ (formula, not a model probability) |
| **Strategy Studio "Top things you can do" (2 suggestions)** | | ✅ (template substitution over the existing deterministic ranking, not generated text) |
| **Leadership view — city geography (lon/lat), product list** | ✅ | |
| **Leadership view — city-opportunity score / band / insight** (`scoreValue`/`bandFor`) | ✅ the *formula* (transparent, deterministic) | ✅ the *inputs* — hand-authored demand/competitor/footprint estimates, not market data |

This table should be kept accurate as real data sources are integrated — move rows up as they become real, and update `PRODUCTION_PLAN.md`'s maturity assessment in the same change.

---

## 5. Leadership view — city-opportunity scoring (`scoreValue` / `bandFor` / `insightFor`)

This is the model behind `leadership-view.html` (the Leadership persona's expansion-opportunity scanner). Like everything else in this document, **it is a deterministic formula, not a trained model** — same input → same output, every time. The map technology around it changed (D3/SVG → Leaflet, see `docs/DESIGN_DECISIONS.md`); this scoring logic was **left unchanged**, and is documented here because it wasn't previously written down anywhere in this project.

**Inputs:** for each of 18 combinations (3 products × 6 cities) a hand-authored entry with three factor values on a 0–100 scale — `demand`, `competitor` (density of rival lenders — *higher is worse*), and `footprint` (existing Piramal presence to build on) — each paired with a short prose `rationale`, plus a hand-set `confidence` label (High/Medium/Low) and a `confidenceNote`.

**Reasoning (the exact formula):**

```
score   = round( demand·0.45  +  (100 − competitor)·0.30  +  footprint·0.25 )
band    = score ≥ 72 → "Strong fit"
          score ≥ 58 → "Promising"
          score ≥ 45 → "Proceed with caution"
          else       → "Not recommended"
```

`competitor` is inverted (`100 − competitor`) so that, like the other two factors, a higher contribution is better. `leadFactorFor` returns whichever of the three weighted terms is largest; `insightFor` renders a band-appropriate one-sentence, plain-English explanation that names that lead factor and its raw value (e.g. *"underlying demand is the strongest signal at 79/100"*). `ACTIONS` maps each band to a recommended next step (Launch / Pilot first / Hold / Deprioritize).

**Outputs:** per city-and-product — a 0–100 `score`, a `band`, a one-line `insight`, a recommended `action`, and (surfaced in the city detail) the three factor values with a Strong/Moderate/Weak level each and their rationales, plus the confidence label/note.

**Business-logic assumptions (explicit):** the weights (demand 0.45 > competition 0.30 > footprint 0.25) encode a plausible prioritisation — realised demand matters most, a crowded market is the next-biggest drag, an existing branch is a helpful-but-smaller tailwind — chosen for narrative plausibility, **not** calibrated against any real launch-outcome data. The band thresholds (72/58/45) are likewise hand-set cutoffs.

**Confidence:** the displayed "High/Medium/Low confidence" per city is a **hand-authored label**, not a model-derived probability — same honesty rule as the Studio's confidence %.

**The critical honesty point — restated plainly:** the **formula above is real** — a transparent, inspectable, deterministic calculation, and every score/band/insight is fully reproducible from the three factor values. But **those three factor values are themselves hand-authored plausibility estimates** for these six specific cities — informed guesses chosen to tell a coherent demo story, **not derived from real demographic, credit-bureau, or competitive-intelligence data**. A judge asking "where does Jaipur's 78 demand come from?" should get exactly that answer: *we authored it as a plausible estimate; the 72/100 score it produces is then a real, transparent computation on top of it.* The realness of the arithmetic must never be used to imply the realness of its inputs.

**Future ML / data path:** replace the hand-authored factor values with real feeds (demand from demographic + internal-portfolio signals, competitor density from branch/location intelligence, footprint from the real branch network already embedded in `branch-pulse-view.html`). The formula, the band/insight/action output shape, and the entire UI were designed so that swap changes only where the three numbers *come from*, not how they're turned into a decision.
