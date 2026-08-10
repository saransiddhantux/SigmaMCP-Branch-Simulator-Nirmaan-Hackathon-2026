# Features

Each feature documented as: purpose, user problem solved, business value, technical implementation, UX decisions, future improvements.

---

## 1. Zone → Cluster → Branch zoom-driven map

**Purpose:** Let a user navigate from national to hyperlocal scale without switching modes or screens.
**User problem solved:** Leadership needs the national picture; a Zonal Head needs their zone; a Branch Head needs one branch. One map should serve all three without separate dashboards.
**Business value:** A single artifact demos to every persona in `docs/IDEA.md`'s target-user table.
**Technical implementation:** Three Leaflet marker layer groups (zone circles, cluster blocks, branch pins), swapped based on `liveMap.getZoom()` in `updateLOD()`. Whichever entity is nearest the map's center at the current zoom drives the right panel (`selectNearestForLOD()`) — see `ARCHITECTURE.md`.
**UX decisions:** Distinct *shapes* per tier (circle / rounded block / pin), not just size, so the hierarchy reads even in a screenshot with no interaction. Panning and clicking a marker both resolve through the same selection function, so behavior is consistent regardless of how the user got there.
**Future improvements:** Real H3 hexagon overlay at the deepest zoom level (the original vision — see `docs/FUTURE_SCOPE.md`); marker clustering/decluttering if the branch count grows well beyond ~1000 pins.

---

## 2. Health scoring & sub-scores

**Purpose:** Reduce a branch/cluster/zone's status to one number, backed by an explainable breakdown.
**User problem solved:** "Is this branch okay?" shouldn't require reading five separate reports.
**Business value:** Consistent scoring language across every level of the org hierarchy.
**Technical implementation:** `buildAutoScorecard()` generates a 0–100 health score plus 4 sub-scores (portfolio quality, micro-market demand, digital adoption, competitive resilience); `buildAggregateD()` averages these across members for Zone/Cluster views. See `docs/AI_DECISIONS.md` for exactly how (and how honestly) this is computed.
**UX decisions:** Same visual (circular gauge + labeled bars) at every hierarchy level, so the eye doesn't have to re-learn the pattern when zooming in or out.
**Future improvements:** Replace the deterministic generator with real aggregated portfolio metrics once a data pipeline exists.

---

## 3. Branch leaderboard

**Purpose:** At Zone/Cluster level, surface which branches need attention without the user hunting through a full list.
**User problem solved:** A Zonal Head managing 100+ branches needs the outliers, not a flat list.
**Business value:** Directly actionable — every row is a one-click drill-in to that branch's full detail.
**Technical implementation:** `paintLeaderboard()` sorts the current level's member branches by health, showing top 3 and bottom 3; clicking a row calls `render(lc)` and (if the live map is active) flies the camera to that branch.
**UX decisions:** Deliberately capped at 3+3 rather than a scrollable full list — keeps the panel scannable at a glance, consistent with the Mudra design system's "progressive disclosure" principle.
**Future improvements:** Configurable threshold/count; trend arrows (improving/declining) once historical data exists.

---

## 4. Micro-market insight

**Purpose:** Pair every branch's raw numbers with one sentence of "so what."
**User problem solved:** Mudra's design rules explicitly call out that a data-heavy component "must answer: what does this number mean for the user" — raw KPIs without narrative don't drive action.
**Business value:** Faster decisions — a Zonal Head reading "retail demand up 18%, a local lipstick-effect signal" acts faster than one reading four unrelated KPI tiles.
**Technical implementation:** A pool of insight templates (`INSIGHT_POOL`) selected deterministically per branch (seeded by the same PRNG as health scores, so it's stable across reloads), each with 2–3 supporting "driver" bullet points revealed on demand.
**UX decisions:** Hidden by default behind a "Why?" disclosure to respect Mudra's "insight text max 2 sentences" rule while still allowing depth for users who want it.
**Future improvements:** Real driver text generated from actual signal deltas rather than a fixed template pool.

---

## 5. AI-ranked recommended actions

**Purpose:** Turn a health score into a next step, not just a diagnosis.
**User problem solved:** Zonal/Branch Heads want to know what to *do*, not only how things look.
**Business value:** This is the feature that makes the tool prescriptive rather than descriptive — a materially different pitch to judges and to real users.
**Technical implementation:** 4 ranked actions per branch, generated alongside the scorecard in `buildAutoScorecard()`. As of 2026-08-09, `rankedActionsForBranch()` draws these directly from the Strategy Studio's own `STRATEGY_CATALOG`, scored with the identical `strategyScore` formula used for the Studio's hero pick — one shared source of truth, so an action's title and its "Simulate →" button always agree on which strategy they mean (previously a separate, disconnected `ACTION_POOL` could show a title with no corresponding Studio strategy at all). The 4 flagship branches keep their hand-authored, narrative-rich action titles, each hand-mapped to its closest real strategy id for the same consistency guarantee.
**UX decisions:** Each action card carries a one-line rationale plus a single "Simulate →" call to action — never more than one primary action per card, per Mudra's button-hierarchy rule. As of 2026-08-09, this card also sits directly under Health in the right rail (was fourth/last) and carries an accent border, so it's the priority read on first scroll rather than something scrolled past to reach.
**Future improvements:** Rank by a real expected-value model instead of a deterministic formula; support dismissing/snoozing an action; richer per-branch variety (auto-generated branches with similar profiles currently surface the same 4 strategies, since it's the same formula — see `docs/AI_DECISIONS.md`).

---

## 6. Strategy Studio — Decision Intelligence Workspace (FLAGSHIP)

**Purpose:** Turn simulation from a slider-calculator into a decision workspace that answers *"given limited budget, what is the smartest intervention?"* — the flagship feature of the product.
**User problem solved:** "Which play should I fund here, and can I defend that choice to leadership?" is normally an offline, multi-week analysis exercise across disconnected spreadsheets.
**Business value:** Demonstrates the "Simulate" stage of the core AI flow (`docs/IDEA.md`) as an executive planning tool — prescriptive, comparative, and defensible, not just a projection. This is the single most judge-impressive surface in the prototype.
**Technical implementation:** A full-screen, map-primary studio (`#studio`) launched from any "Simulate →" or "+ New simulation" entry point. The single live Leaflet map is re-parented in and participates (coverage-radius circle, glowing affected-branch markers, pulsing ROI badge). A deterministic economics engine (`computeStrategyProjection`) produces 9 live KPIs; `recommendStrategy` picks the hero; `generateStrategyOptions` builds the 3-way trade-off. See `ARCHITECTURE.md` and `docs/AI_DECISIONS.md §3a` for the exact formula and its honesty framing.
**The sections (updated 2026-08-09):** hero recommendation (ROI/confidence/investment/risk/payback, collapsible, always reflecting the active selection); (1) 7 rich strategy cards (icon, description, ROI, difficulty, confidence); (2) rollout config — budget slider (₹5L–₹50L), timeline (3/6/12mo), deployment scope (Pilot/Cluster/Zone/State), target geography; (3) a **Simulate CTA**, disabled until the user touches a strategy or config control, then enabled and relabelled "Simulate Now"; (4) once clicked, a **"Top things you can do"** callout with exactly 2 concrete action suggestions; (5) live 9-metric business projection with animated counters and change-flash; (6) 3-way trade-off with Best ROI / Fastest Payback / Lowest Risk / Highest Growth badges; (7) expandable AI-reasoning accordion with per-statement confidence; (8) executive summary (justification, top risks, key assumptions, timeline); (9) approval bar — Approve / Save scenario / Compare later / Export business case. Sections 5–9 (formerly 4–8) are hidden until Simulate Now first runs, then stay visible and stay live for the rest of the session.
**UX decisions:** Map stays visible and *participates* (never covered — the earlier modal-over-map bug is gone). Once past the Simulate gate, everything recomputes live on any control change — no "recalculate" step. Light Mudra surface, solid elevated panel (no glassmorphism), premium via spacing/typography/motion (see `docs/DESIGN_DECISIONS.md`). The gate itself is a deliberate progressive-disclosure choice: lead with a direct 2-suggestion answer before the supporting analytics, rather than showing all nine sections at once on open.
**Future improvements:** Replace the deterministic engine (and the 2-suggestion template layer) with a real trained uplift/demand model (Layer 4); persist approved/saved scenarios to a real backend instead of `localStorage`; render a true catchment heatmap.

---

## 7. "My Simulations" — history navigation panel

**Purpose:** Let a user get back to any simulation they've already tried or approved without reconstructing it from scratch.
**User problem solved:** A Zonal Head comparing interventions across several branches in one session previously had no way back to an earlier configuration once they navigated away or closed the Studio — direct request: "it should have the simulation history which I have already tried."
**Business value:** Turns the Studio from a single-shot calculator into a genuine workspace for comparing multiple considered options over a session — closer to how a real planning exercise actually happens (try several things, come back to the best one).
**Technical implementation:** A second floating panel (`#studio-history`), docked left as a mirror of the existing right-docked `#studio-panel`, both overlaying the same full-bleed map. Every distinct "Simulate Now" click logs one entry (`logSimHistory`, tagged `'tried'`); every approval logs a second, separately-tagged entry (`'approved'`) — both persisted to `localStorage` (`bis_sim_history`, capped at 30) so history survives a reload. Clicking an entry (`restoreScenario()`) re-establishes that exact branch, strategy, and rollout config and reveals results immediately, whether the Studio is already open on a different branch or closed entirely.
**UX decisions:** Reuses the exact same solid-card visual treatment as the main workspace panel — no new visual language for "this is also a panel." History entries show a status badge (Tried / Approved), relative time, strategy, branch, and headline stats (ROI/confidence/investment) so the list is scannable without opening each one. Deliberately logs once per genuine open→simulated transition, not per slider tweak, so the list stays a meaningful record rather than a noisy activity log; restoring a past entry does not itself log a new one.
**Future improvements:** Let a user delete/pin individual history entries; group by branch; surface a diff between two selected history entries (what changed, and what it did to the projection).

---

## 8. Map participation during simulation

**Purpose:** Show a strategy's projected reach and impact *where it will actually happen*, live, as parameters change.
**User problem solved:** A number in a panel is abstract; a coverage radius that grows as you widen deployment, and a pulsing ROI badge on the branch, make the decision spatial and concrete.
**Business value:** A strong, memorable demo moment — directly requested as "the map should participate... coverage radius expands, branches glow, markers pulse."
**Technical implementation:** `updateStudioMap()` draws an `L.circle` coverage radius scaled by deployment scope + budget, a `studioHighlightLayer` of `L.circleMarker`s for affected branches (colored by projected uplift), and a pulsing ROI-badge marker on the selected branch — all rebuilt on any config/strategy change. A dedicated layer group avoids fighting the existing zoom-LOD marker system.
**UX decisions:** Non-interactive overlays (don't intercept clicks) so they never block map interaction. "Heatmap" is graduated coverage + colored markers, not a raster plugin (see `docs/DESIGN_DECISIONS.md`).
**Note:** The older, dormant inline simulation's `previewSimOnMap()` (a single pulsing pin) is retained but superseded by this richer participation model.

---

## 9. Proactive alert banner

**Purpose:** Surface at-risk branches without the user having to go looking for them.
**User problem solved:** Per `docs/IDEA.md`'s original narrative, teams currently find out about problems reactively (quarterly reviews) rather than proactively.
**Business value:** Directly demonstrates the "non-spam, interval-based alerting" requirement from the original product brief.
**Technical implementation:** Conditionally rendered in `paintHealthCard()` whenever the selected branch's dossier has a non-null `alert` field; auto-generated branches get one whenever their health score falls below 48.
**UX decisions:** Dismissible, with a "View reason" disclosure rather than dumping the full explanation immediately — consistent with Mudra's guidance-message anatomy (short title, one disclosure, one action).
**Future improvements:** Real push notification delivery (the original brief's mobile alerting requirement) — not implemented in this frontend-only prototype at all.

---

# Leadership View (`leadership-view.html`)

A separate single-file build for the Leadership persona (`docs/IDEA.md`). React 18 + Leaflet (both via CDN, no build step), reusing the same Mudra 2.0 tokens as the Zonal Head view. Where the Zonal Head view is branch-operations first, this one is a **market/product expansion-opportunity scanner**: pick a product, see which of six cities is the best next move, compare a shortlist, decide.

## 10. National product-opportunity map

**Purpose:** Show, for one product at a time, which cities are the strongest expansion/launch opportunities — spatially, on a real map.
**User problem solved:** A product-strategy lead deciding "where do we launch/expand this product next?" has no single view tying demand, competition, and existing footprint to geography; the answer is assembled by hand.
**Business value:** Turns a scattered, multi-spreadsheet expansion call into a one-glance, per-product ranked read of the national candidate set.
**Technical implementation:** A live Leaflet 1.9.4 + OpenStreetMap map (same library, CDN URL and tile source as `branch-pulse-view.html`, no API key/signup). Because `react-leaflet` needs a build step — incompatible with this CDN-only project — the vanilla Leaflet API is driven imperatively from `IndiaHexMap` via a `useRef` + `useEffect` pattern: React renders one empty `<div>`, an init `useEffect` (empty deps) creates `L.map` once, and a second `useEffect` clears + re-adds a marker `L.layerGroup` whenever the product's entries / selection / biggest-opportunity change. Six `L.divIcon` city markers sit at real lon/lat, coloured by `BAND_MAP_COLOR[band]` for the selected product; the biggest opportunity pulses; the selected marker gets an outline and a permanent tooltip. Marker clicks call the existing `onSelectCity` (App state model unchanged). This **replaced** the original static D3/SVG hex-grid illustration (`docs/DESIGN_DECISIONS.md`).
**UX decisions:** Three floating "pill" overlays (top-left brand/back, top-right product selector, bottom-left band legend) float over the live basemap; the map host is given its own stacking context (`z-index:1`) so Leaflet's internal panes/controls stay trapped below them — the same z-index fix `branch-pulse-view.html` uses on `#studio-map`. Zoom control is docked bottom-right to avoid colliding with the pills. Native `bindTooltip` renders the hover card (city, state · tier, score, band badge, action) so its content matches the previous overlay exactly. A simple, honest loading/error state covers the pane until the map is ready.
**Future improvements:** Optional instant-load offline SVG fallback that upgrades to the live map (as `branch-pulse-view.html` ships) — deliberately skipped here for scope, so a slow/blocked CDN shows a loading state rather than a fallback map; marker clustering only becomes relevant well beyond six cities.

## 11. City-opportunity scoring & plain-English insight

**Purpose:** Reduce each city-for-a-product to one explainable score, a band, and a one-sentence "why".
**User problem solved:** A ranked list is only actionable if the user can defend *why* a city ranks where it does — a bare number can't be taken into a planning conversation.
**Business value:** Consistent, transparent scoring language across all 18 (3 products × 6 cities) combinations; every recommendation is auditable down to the factor that drove it.
**Technical implementation:** `scoreValue` = `demand·0.45 + (100 − competitor)·0.3 + footprint·0.25` (rounded); `bandFor` buckets into Strong fit / Promising / Proceed with caution / Not recommended; `leadFactorFor` picks the largest weighted contributor and `insightFor` renders a band-appropriate sentence citing it; `ACTIONS` maps each band to a recommended next step. See `docs/AI_DECISIONS.md §5` for the exact formula and its honesty framing. **This logic is deliberately left unchanged** by the map swap — it is already a good, explainable, deterministic (non-ML) model in the same spirit as the Zonal Head view's scoring.
**UX decisions:** The lead-factor phrasing is specific ("underlying demand is the strongest signal at 79/100" / "competitor density sits at 62/100, a real headwind"), never a vague "the AI thinks…". Each factor also shows its raw 0–100 value, a Strong/Moderate/Weak level and a one-line rationale, so the score is fully unpacked.
**Future improvements:** Replace the hand-authored per-city demand/competitor/footprint inputs with real demographic/competitive-intelligence feeds — the formula and output shape are designed to survive that swap unchanged (`docs/AI_DECISIONS.md §5`).

## 12. Multi-city comparison

**Purpose:** Let the user place a shortlist of candidate cities side by side before committing.
**User problem solved:** Expansion decisions are comparative ("Jaipur vs Indore vs Nagpur for Gold Loan?"), not one-city-at-a-time; a ranked list alone doesn't make the trade-offs between a few close contenders easy to read.
**Business value:** Surfaces the trade-off a launch decision actually turns on — two cities can share a band yet differ sharply on which factor carries them.
**Technical implementation:** `App` holds `compareCityIds` (max 3); `CityDetail`'s add/remove toggle feeds it; a side-pane footer tracks the count and opens `CompareModal`, which renders `CompareView` — one `CompareCard` per city showing all three factors and the score, with the strongest of the set flagged. Escape/backdrop close the modal.
**UX decisions:** Capped at 3 to keep the comparison scannable (same "progressive disclosure" spirit as the Zonal Head leaderboard's 3+3 cap). The compare set is pinned to the currently-selected product, so a comparison is always apples-to-apples.
**Future improvements:** Compare the same city across *products* (not just cities within one product); export the comparison as a one-pager for a planning deck.
