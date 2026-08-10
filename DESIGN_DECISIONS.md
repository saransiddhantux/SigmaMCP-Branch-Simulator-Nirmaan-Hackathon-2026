# Design Decisions

Each entry: what changed, why, expected UX improvement, trade-offs accepted. Newest at the bottom.

---

## Mudra 2.0 design system adoption

**What changed:** Built entirely on Piramal's Mudra 2.0 token system (semantic color/spacing/radius/shadow tokens, Nunito typeface, documented component anatomy) rather than ad hoc styling.
**Why:** On-brand from the first screen; Mudra also encodes real financial-UX rules (tabular numerals for money, one primary action per section, insight-layer requirements) that improve the product, not just its look.
**Expected UX improvement:** Consistent, accessible-by-default (WCAG 2.1 AA baseline), and immediately recognizable as a Piramal product.
**Trade-offs:** Some Mudra rules (e.g., "max 2 guidance prompts per view") constrained how much could be shown at once — accepted deliberately in favor of not overwhelming the user.

---

## Modal simulation → inline panel

**What changed:** The what-if simulator originally ran in a centered, full-screen-dimming modal dialog. It now runs inline, replacing the Actions card's content in place.
**Why:** Direct bug report — the modal visually covered the map while simulating, defeating the point of a map-first tool and breaking the "preview on the map" requirement entirely.
**Expected UX improvement:** Map and simulation results are visible **side by side** at all times; a "← Back" link returns to the action list without ever losing map context.
**Trade-offs:** Less visual emphasis/drama than a full-screen modal; mitigated by the map-pin preview effect taking over that "moment" role instead.

---

## Map-pin simulation preview

**What changed:** Running a simulation now shows a live pulsing ring + projected-impact badge directly on the affected branch's map pin.
**Why:** Explicit request to make simulation results visible spatially, not just numerically.
**Expected UX improvement:** Turns an abstract percentage into a concrete, memorable, spatially-grounded moment — also a strong demo beat.
**Trade-offs:** Only implemented for the live (Leaflet) map — the offline SVG fallback has no equivalent, since it lacks per-branch pin placement at all (see below).

---

## Mappls → Leaflet + OpenStreetMap

**What changed:** The live interactive map first integrated Mappls' Web SDK (India-specific, Survey-of-India-compliant mapping). It was replaced with Leaflet.js + OpenStreetMap tiles.
**Why:** Mappls requires account signup and an API key; real access/signup issues were hit during the hackathon. Leaflet + OSM needs neither, eliminating an entire class of "it works on my machine but not the judges'" risk.
**Expected UX improvement:** Live map now activates reliably for anyone, with no setup step, ever.
**Trade-offs:** Lost Mappls' India-specific address/place-name precision and any BFSI-compliance argument for using a Survey-of-India-aligned provider — acceptable for a demo; worth revisiting per `docs/FUTURE_SCOPE.md` if this goes further.

---

## Flat state choropleth → Zone/Cluster/Branch zoom hierarchy

**What changed:** The live map originally colored whole Indian states by an aggregate health score (a state-level choropleth using a public state-boundary dataset). It was replaced with three zoom-tiered marker layers (zone circles → cluster blocks → branch pins) built directly from real branch coordinates.
**Why:** Explicit request to reflect the real Zonal → Cluster → Branch organizational hierarchy, and to populate the map with the real branch network rather than 4 hand-picked flagship branches.
**Expected UX improvement:** The map now mirrors how the business actually organizes itself, and "zoom in" has real meaning (more branches become visible and selectable) rather than just visual detail.
**Trade-offs:** Dropped the topojson state-boundary layer and its dependency entirely — a deliberate simplification (fewer network dependencies, faster live-map init) at the cost of state-boundary outlines. Clusters are algorithmically derived (k-means on lat/lon, ~7 branches/cluster) since no official cluster field exists in the source data — clearly labeled as such rather than presented as ground truth.

---

## Distinct shapes per zoom tier (circle / block / pin)

**What changed:** Zones render as circles, clusters as rounded square "blocks," branches as small pins — three different shapes, not just three sizes of the same shape.
**Why:** Explicit request; also a genuine UX improvement — shape is legible even in a static screenshot, whereas size-only differentiation requires side-by-side comparison to read.
**Expected UX improvement:** The hierarchy is instantly scannable, which matters a lot for a hackathon judge glancing at a screen for a few seconds.
**Trade-offs:** None significant — implemented via one shared `divIcon()` factory with a CSS class switch, so it added negligible complexity.

---

## Offline map kept intentionally limited in scope

**What changed:** Rather than trying to make the offline (no-internet) fallback support all 733 real branches, it still only covers the original 4 flagship branches on a flat state choropleth.
**Why:** The 733 real branches' coordinates are real lat/lon; the offline map is a flat custom SVG projection (from a third-party India map file) with no reliable inverse-projection formula to place precise pins on it. Getting this exactly right was judged lower-value than investing that time in the live-map experience, which is demonstrably reliable (Leaflet + OSM, no signup).
**Expected UX improvement:** N/A — this is a scope-limiting decision, not a UX improvement.
**Trade-offs:** A judge or user without internet access sees a materially less impressive experience. Flagged explicitly in `SETUP.md`'s Known Issues and `PRODUCTION_PLAN.md` rather than left as a silent gap.

---

## Simulation → "Strategy Studio" (Decision Intelligence Workspace)

**What changed:** The what-if simulation moved from a cramped inline panel in the 452px right column to a full-screen, map-primary **Strategy Studio**: the live map fills the surface, and a tall opaque workspace card floats docked-right with hero recommendation → strategy cards → rollout config → live KPI grid → 3-way trade-off → AI reasoning → executive summary → approval bar.
**Why:** The brief asked to make simulation the flagship feature — an "AI Strategy Studio / Executive Planning Workspace," reframing the core question from "what happens if I move a slider?" to "given limited budget, what's the smartest intervention?". The 8-section spec simply doesn't fit a 452px column.
**Expected UX improvement:** The decision gets the room and gravitas it deserves; the map stays visible and *participates*; comparison and reasoning are first-class rather than afterthoughts.
**Trade-offs:** More surface area to maintain; the map is re-parented between dashboard and studio (one instance, `invalidateSize` on each move) — a deliberate choice over a second map instance to avoid state divergence, at the cost of a small re-parenting complexity (documented in `ARCHITECTURE.md`).

**Sub-decisions (confirmed with the user):**
- **Light studio, matched to the dashboard** — the brief said "preserve dark/light theme support," but the app has *no* theme system to preserve (verified: zero `prefers-color-scheme`/`data-theme`/`.dark`). Rather than build an app-wide theme system (out of scope for "rebuild only the simulation"), the studio stays on the existing light Mudra surface and earns "premium" through spacing, typography, hierarchy, and subtle motion — which is what the brief's own Visual Design section emphasizes.
- **Map-primary with a floating panel** (user's pick) over a full-width map-left/workspace-right split. Trade-off accepted: the narrow panel makes the 3-way comparison tight, so the trade-off options are **stacked vertically** (with winner badges) rather than shown as 3 cramped side-by-side columns.
- **No glassmorphism, no gradients** (brief's explicit ban) — the floating panel is a solid `--bg-surface` card elevated purely with `--sh-lg`.
- **"Heatmap" realized as coverage circles + colored affected-branch markers**, not a raster heatmap — a deliberate choice to avoid adding an unproven `leaflet.heat` plugin dependency. Stated honestly rather than claiming a capability that isn't there.

**Notable implementation fix (worth recording):** the floating panel initially rendered *behind* the map because Leaflet's internal map panes use z-indexes up to ~700, and `#studio-map` wasn't establishing its own stacking context — so those high z-indexes competed directly with the panel. Fixed by giving `#studio-map` `z-index:1` (creating a stacking context that traps Leaflet's internals below the panel's `z-index:2`). A good reminder that third-party map libraries bring their own z-index scale into any shared stacking context.

---

## Hero recommendation icon size

**What changed:** The Strategy Studio's "AI-Recommended Strategy" icon shrank from a card-dominating, browser-default-sized SVG down to 16×16px, sitting inline on one line with its label.
**Why:** The icon SVG (`iconSvg()`) was never given explicit `width`/`height`, so it fell back to the browser's default replaced-element size — every other icon usage in the file sets its size explicitly (either inline or via a scoped CSS rule), and this one was simply missed.
**Expected UX improvement:** The hero card now reads as a compact, confident label + headline, not a full-bleed graphic — restoring the intended visual hierarchy (title and rationale are now what draws the eye).
**Trade-offs:** None — this was an unambiguous sizing bug, not a design trade-off.

## KPI cards relocated under the Health card; branch/zone/cluster name surfaced

**What changed:** The 3 KPI trend cards (RM Contribution, Disbursement Mix, Competitor Density) moved from the left column (under the map) to directly beneath the Health card in the right rail, and now stack in a single column rather than 3-across. The Health card also gained a prominent name line (branch name at branch level; zone/cluster name at those aggregate levels) above the gauge.
**Why:** Visual review feedback — the KPIs read as disconnected from the health context they explain, and the health card had no identity of its own without cross-referencing the topbar breadcrumb.
**Expected UX improvement:** Health score, sub-scores, and the KPIs that explain *why* now form one contiguous reading block in the right rail; the card is self-describing without needing the breadcrumb.
**Trade-offs:** Single-column KPI cards take more vertical scroll than the old 3-across layout did in the wider left column — accepted, since a cramped 3-up grid at 452px width would have looked worse than a comfortable single column. `.card + .card`'s automatic sibling margin no longer reaches the Actions card (a `.kpi-row`, not a `.card`, now sits between them), so `.kpi-row` was given explicit `margin-top`/`margin-bottom` to preserve consistent 24px spacing.

> **Superseded below** — the "more vertical scroll, accepted" call turned out wrong in practice: the very next round of feedback was that this block pushed Recommended Actions off the first screen. Recorded here rather than deleted, since the failed trade-off is itself the useful signal.

## KPI row: single-column → compact horizontal strip

**What changed:** Reversed the single-column decision above. The KPI row is horizontal again (3 columns), but each tile is now genuinely compact — smaller type throughout, ellipsis-truncated labels, and the sparkline chart dropped entirely. Net height ~79px, down from ~450px+ stacked.
**Why:** Direct feedback that the vertical stack was "taking too much space," pushing Recommended Actions below the first scroll — the very trade-off the prior decision had guessed would be acceptable. It wasn't.
**Expected UX improvement:** Recommended Actions — the primary action surface of the whole app — is now visible without scrolling on typical viewport heights. The KPI strip becomes a true "quick glance," not a detail view.
**Trade-offs:** Dropped the sparkline trend charts from this location (kept for the Strategy Studio's projection panel, which has room for them). Long values (e.g. "4 rivals ≤3 km") occasionally wrap to two lines in the narrowest tile rather than overflowing — accepted, since the label truncates safely via ellipsis but a numeric value should never be clipped.
**Lesson:** When a stated trade-off is really a guess about user tolerance for scroll depth, it's worth flagging as a guess (not a settled decision) until it's actually been seen on screen — this one didn't survive first contact.

---

## Micro-market insight moved above the map, and made crisper

**What changed:** The insight card is now the first thing in the left column, above the map (was below it). Its internal padding, icon, and type sizes were all reduced.
**Why:** Feedback that a "so what" narrative deserves to lead, not trail, the map — and that its old size (16px title, 16px icon, 16px padding) was too generous for what's now a top-of-page element.
**Expected UX improvement:** The single most decision-relevant sentence on the page is now the first thing read, not something scrolled past below a large map.
**Trade-offs:** None identified — this is a straightforward reprioritization plus a density pass consistent with the KPI-row compaction above.

## "+ New simulation" promoted to a primary button

**What changed:** Changed from `.btn-ghost` (transparent, body-colored text) to `.btn-primary` (solid orange).
**Why:** Direct feedback that the button was "not visible right now" — a ghost button at `btn-sm` size, sitting next to an equally low-key "AI-ranked" pill, didn't read as an actionable CTA at all.
**Expected UX improvement:** A user who wants to run a from-scratch simulation (not tied to one of the 4 ranked actions) can now actually find the entry point.
**Trade-offs:** None — Mudra's "max one primary button per section" rule is still respected (the action-card list below uses `.btn-secondary` for its own "Simulate →" buttons).

## Strategy Studio hero card: collapsible, and reflects the active selection

**What changed:** Added a collapse/expand chevron, and rebuilt the hero card to always describe whichever strategy is currently selected — not permanently the AI's original top pick. When the active selection differs from the AI's pick, the badge reads "Selected strategy" (not "AI-recommended") and a "AI recommends X instead — use it →" link appears.
**Why:** Two related pieces of feedback: the hero card felt "too in the face" taking up a lot of vertical space with no way to shrink it, and — the more important functional gap — selecting a different strategy card gave no visible confirmation at the top of the workspace of what was now active, since the hero never changed.
**Expected UX improvement:** The hero card now does double duty as both the AI's opening pitch *and* a persistent "what am I currently looking at" indicator, collapsible once its job is done. Also fixed two related copy bugs this surfaced: `heroRationale()` and the executive summary's "Recommended strategy" label were both unconditionally claiming "highest risk-adjusted return," which was already inaccurate whenever a user had manually selected a non-optimal strategy in the executive summary — both now say "Selected strategy" / "you're testing X" when that's what's actually true.
**Trade-offs:** The CTA button's meaning changed from "select the AI's pick" (`Test this strategy`) to "go compare strategies" (`Compare other strategies ↓`), since the hero no longer needs a button to *make* a selection — it's always already showing the active one. A returning user who remembers the old button's behavior may need a moment to notice the relabel.

---

> **Note on the entries below:** first applied to `~/Downloads/branch-pulse-view copy.html`, a copy that had diverged from this folder's canonical `branch-pulse-view.html` (it contains a Market News feed and a Nearby-branch-comparison card built outside this project's documented history). That copy has since been synced back as this folder's canonical file — see `CHANGELOG.md`'s 2026-08-09 entries for the full divergence-and-resolution history. The entries below describe the current canonical file.

## Market News & Product Impact section removed

**What changed:** Deleted the `#news-card` block entirely — an illustrative news feed with per-item product-impact chips, which also contained a second, duplicate copy of the branch's micro-market insight nested inside it.
**Why:** Direct feedback to remove it.
**Expected UX improvement:** One fewer card between the map and the right rail; removes a duplicate rendering of content (`#insight-slot` above the map) that was already shown once.
**Trade-offs:** The illustrative news→product-impact mapping was a reasonable idea for a future real news-feed integration (see the old `NEWS_ITEMS` comment: "a production build would pull this from a live news/market-data feed") — that concept isn't preserved anywhere else. If it's wanted again later, it's fully recoverable from this changelog entry, not silently lost.

> **Corrected below, same day** — this went further than intended. The actual feedback was to remove two specific callout boxes inside the card, not the whole card. Recorded here rather than deleted, per this project's honesty-over-tidiness convention for reversed decisions.

## Market News: restored, minus the disclaimer + duplicate-insight boxes only

**What changed:** Re-added `#news-card` (header, area chip, product filters, news list) but permanently left out the two elements actually being objected to: the "Sample data for this prototype" disclaimer box and the duplicate embedded micro-market insight box.
**Why:** Direct correction: "I just asked to remove the 2 blue boxed and keep rest of the info."
**Expected UX improvement:** The news feed's actual content (regulatory/market items mapped to specific products) is back, without the two elements that were either redundant (the insight, already shown once above the map) or meta-commentary not meant for a live demo view (the disclaimer).
**Trade-offs:** The disclaimer's honesty framing ("Sample data for this prototype... a production build would pull this from a live feed") no longer appears inline in the UI. That honesty is preserved instead in `docs/AI_DECISIONS.md` and the demo script, which is where a technical judge would actually look for it — the inline banner was arguably redundant with the card's own "Illustrative feed" badge, which stayed.

## Recommended Actions promoted above Nearby/KPI cards

**What changed:** The right rail order changed from Health → Nearby comparison → KPI row → Actions to Health → **Actions** → Nearby comparison → KPI row. The Actions card also got an accent border matching `--border-primary` (the same token the Studio's hero card border uses) to read as the priority card.
**Why:** Feedback that Actions needed a different treatment to be visible on first scroll — three cards deep was too far down.
**Expected UX improvement:** The primary action surface in the whole right rail now needs only one card (Health) scrolled past, not three, on typical viewport heights.
**Trade-offs:** Diagnostic detail (nearby-branch comparison, the 3 trend KPIs) now sits *below* the prescriptive actions instead of building up to them — a deliberate inversion, since "what should I do" is a stronger first read than "here's more context," and a user who wants the context can still get it one scroll down.

## Strategy Studio: gated "Simulate Now" CTA + top-2 action-suggestion callout

**What changed:** The Studio used to show everything at once — hero, strategy cards, rollout config, live KPIs, trade-off, reasoning, summary, and the approval footer, all live-updating from the moment it opened. It now opens showing only strategy selection and rollout config, with a disabled CTA ("Choose a strategy to simulate"). The first time the user touches any control, the CTA enables and relabels to "Simulate Now." Clicking it reveals a new headline callout — exactly 2 concrete, numbered action suggestions — followed by everything that used to be visible immediately (KPI grid, trade-off, reasoning, summary, footer).
**Why:** Feedback that the simulation side should show just its initial parameters first, gate the call-to-action until they're set, and lead with a direct "here's what to do" answer rather than a full dashboard on first paint.
**Expected UX improvement:** A first-time user gets a simple, linear path (configure → simulate → here are your 2 options) instead of nine sections of live-updating detail competing for attention at once; the detail is still one click away for anyone who wants to dig in, not removed.
**Trade-offs:** Every Studio control already carries a sensible default (there's no real "empty" state), so "parameters added" had to be approximated as "the user touched something" rather than "every field is non-empty" — an honest proxy, but not a literal validation of completeness. The first-open experience is now slightly less impressive at a glance (less visible at once) in exchange for being less overwhelming — accepted, since the deep-dive is one click away, not gone.
**Visual language reused, not invented:** the suggestions callout reuses the same tinted `--border-primary`/`--bg-primary-light` treatment as the hero card, so it reads as "the AI's headline answer" using a pattern the user has already seen in this workspace, rather than introducing a fourth visual style for "important callout."

## Recommended Actions unified with the Strategy Studio's strategy catalog

**What changed:** The dashboard's 4 ranked action cards no longer come from a separate, disconnected list — for the ~729 auto-generated branches they're now the top 4 entries of the exact same `STRATEGY_CATALOG` the Studio itself ranks, and every "Simulate →" click opens the Studio pre-selected to that specific strategy.
**Why:** Direct feedback that clicking an action and landing in the Studio felt disconnected — "the recommended options are not in sync with what's inside the page."
**Expected UX improvement:** What you click is what you get. A user who reads "Increase RM Capacity" and clicks Simulate now genuinely opens a workspace already configured around RM capacity, not an unrelated AI pick.
**Trade-offs:** The 4 hand-authored hero branches keep their bespoke, narrative-specific titles (a deliberate, documented investment — see `docs/FEATURES.md` §5) rather than being flattened to generic strategy names, so they were instead hand-mapped to their *closest* real strategy id. This is an honest best-effort mapping ("Add 2 feet-on-street RMs" → Increase RM Capacity), not a perfect one — a couple of the mappings (e.g. "Upsell wealth products" → Cross-sell LAP, since no wealth strategy exists in the 7-item catalog) are approximate by necessity. Documented rather than silently glossed over.

## "My Simulations" — left-docked navigation panel

**What changed:** Added a second floating panel, docked left, on the opposite side of the Studio's map from the existing right-docked workspace panel — a running, persistent list of every simulation tried or approved, each one clickable to jump straight back into that exact configuration.
**Why:** Direct request for a navigation panel showing "the simulation history which I have already tried."
**Expected UX improvement:** Nothing tried is ever lost to a closed tab or an accidental exit — a Zonal Head comparing several branches' interventions across a session can always get back to an earlier one in one click, without re-configuring it from scratch.
**Trade-offs:** Two floating panels plus the map now share the same fixed-size viewport with no responsive redesign below ~1024px (an existing, already-documented limitation of the map-primary layout, not a new one this introduces). Chose to log one history entry per genuine "Simulate Now" transition rather than per slider tweak, which means mid-session parameter exploration before that first click isn't captured — an intentional choice to keep the list meaningful rather than a noisy log of every intermediate drag.

---

## Leadership view: D3/SVG hex-grid map → Leaflet + OpenStreetMap

**Context:** applies to `leadership-view.html` (the Leadership persona's view — see `docs/USER_JOURNEY.md`). Its map originally rendered a **static, decorative D3 `geoMercator` hex-grid** illustration of India (a `buildHexGrid`/`HEX_GRID` texture projected from an embedded `INDIA_MAINLAND` GeoJSON), with the six city pins absolutely positioned on top via the same projection — no pan, no zoom, no real basemap, roughly analogous to `branch-pulse-view.html`'s offline SVG fallback rather than its live map.

**What changed:** Replaced that with a real, live, pannable/zoomable **Leaflet 1.9.4 + OpenStreetMap** map — the exact same library, CDN URL and tile source as `branch-pulse-view.html`, for genuine cross-view consistency (not a different "similar" library). Because `react-leaflet` requires a build step (incompatible with this CDN-only, no-bundler project), the vanilla Leaflet API is driven imperatively from the `IndiaHexMap` React component: a plain `<div ref>` that React renders once and Leaflet then owns, initialised in a mount-only `useEffect`, with the six markers rebuilt in a second `useEffect` (clear + re-add a layer group) whenever the selected product's entries / selection / biggest-opportunity change. The dead D3 code, the embedded GeoJSON, and the D3 CDN `<script>` were all removed once nothing referenced them (verified by grep).

**Why:** The static illustration couldn't do what a national opportunity-scanner actually wants — pan/zoom to inspect real geography — and shipping two different map technologies across the two persona views was an unnecessary inconsistency. Reusing `branch-pulse-view.html`'s proven Leaflet+OSM stack (no API key, no signup — the same reason it beat Mappls, see the entry above) means one map story for the whole product.

**Expected UX improvement:** The Leadership map is now a genuine map — real place context, pan, zoom — with the six cities as band-coloured markers on it, and the biggest opportunity pulsing. Marker hover/click behaviour and the floating pill UI (brand/back, product selector, legend) are preserved exactly; they now float over a live basemap instead of a flat SVG.

**Trade-offs:** Adds a runtime dependency on the Leaflet CDN + OSM tiles (mitigated by loading Leaflet synchronously in `<head>` so `L` exists before the app boots, and by an honest loading/error state in the map pane). **Deliberately did not** replicate `branch-pulse-view.html`'s instant-load offline-SVG-then-upgrade dual-mode robustness — a stated scope decision: a slow or blocked CDN shows a "Loading live map…" (or error) message rather than a fallback map. Colouring moved from the hex *cells* (old design: red pins over band-coloured cells) onto the *markers themselves*, since there's no longer a grid behind them — this matches the task's intended "markers coloured by band" model and, if anything, reads more directly.

**Notable implementation note (same lesson as the Studio):** Leaflet's internal panes/controls use z-indexes up to ~1000, which would render over the absolutely-positioned floating pills. Fixed by giving the Leaflet host (`.leaflet-map-host`) its own stacking context (`z-index:1`) so those internals stay trapped below the pills (`.map-pill`, `z-index:10`) — the identical fix `#studio-map` uses in `branch-pulse-view.html`. The zoom control was also moved to bottom-right so it never sits under the top-left/top-right pills.

**Verified (headless, scripted interaction test — same pattern as every other change here):** served locally, driven with headless Chrome `--dump-dom`. 22/22 checks passed with zero console errors: Leaflet container initialises; six band-coloured markers render; the biggest-opportunity pulse correctly moves when the product changes (Jaipur for Gold Loan → Indore for Personal Loan); clicking a marker renders `CityDetail`; adding two cities opens the comparison modal with two cards.

---

## Leadership view: visual-consistency reconciliation against `branch-pulse-view.html`

**What changed:** A pass to confirm `leadership-view.html`'s Mudra 2.0 styling matches the more mature `branch-pulse-view.html` (the shipped reference), reconciling any genuine drift rather than doing a wholesale rewrite. Finding: the two files' token **values** are already identical — `leadership-view.html` just structures them as primitives → semantics (`--orange-100` → `--bg-primary`) where `branch-pulse-view.html` uses flat shorthand (`--bg-primary`), and every colour / radius / shadow / spacing value resolves to the same hex/px. The one real drift found and fixed: `.mudra-btn` carried a base `min-height: 44px` that silently overrode its own size tokens (`--sm` 32px, `--md` 40px), forcing **every** button to ≥44px — whereas `branch-pulse-view.html`'s `.btn` scale is 40px (`.btn`) / 32px (`.btn-sm`). Removed the base `min-height` so the size tokens take effect and match.

**Why:** Both views ship as one product under one design system; a button height that differs between them (and that defeated the file's own size scale) is exactly the kind of drift a consistency pass should catch.

**Expected UX improvement:** Buttons in the Leadership view now match the Zonal Head view's button scale precisely; the primary/secondary/sm sizing behaves as the token names imply.

**Trade-offs:** Every button in the Leadership view is now up to 4px shorter than before. This is a visible change to a file that already "worked", and 44px is a common accessibility tap-target minimum — but the reference view uses 40px and the 44px base was overriding the file's own documented size tokens, so reconciling to the reference (and letting the size scale work) was the right call. Deliberately did **not** rename `leadership-view.html`'s primitive→semantic token architecture to `branch-pulse-view.html`'s shorthand: the values already match, renaming would be a large, risky, purely-cosmetic churn with no visual effect, and the extra primitive layer is arguably the more complete token system.
