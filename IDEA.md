# The Idea

## Business problem

Piramal Finance's business, strategy, product, distribution, and branch-expansion teams make growth decisions using highly fragmented datasets — existing branch performance, demographics, demand signals, competition, geography, credit behavior, and local economic indicators — with no single hyperlocal view tying them together. Forward-looking questions ("where should we open a branch," "where should we pilot the Saarthi Fleet Program," "is a competitor about to eat our market share in this catchment") are slow to answer and typically reactive — teams find out a branch is underperforming at the next quarterly review, not when it starts happening.

## Original concept

Divide India into a continuous grid of hyperlocal micro-markets, each combining internal business data with external demographic, geographic, and economic signals into a live picture. A user selects any micro-market to see:

- **Current health** — active product performance, demand, portfolio quality, customer behavior
- **Product opportunity** — which existing products would perform well here
- **Product experimentation** — where to pilot a new product before scaling it
- **Branch simulation** — predicted customer base and business potential if a physical branch opened here
- **Expansion opportunity** — a look-alike analysis matching underserved areas to the profile of the best-performing existing markets

The originally conceived geographic unit was a **~3km H3 hexagon grid** (see `architecture_flowchart.png` for the full target-state design). The prototype instead organizes around the real **Zone → Cluster → Branch** organizational hierarchy, using real branch coordinates rather than a synthetic hex grid — a deliberate scope decision for the hackathon timeline, explained in `docs/DESIGN_DECISIONS.md`. The hex-grid vision remains the long-term target; see `docs/FUTURE_SCOPE.md`.

## Core AI flow

The intended pipeline, unchanged from the original architecture:

```
Geo + Business Data → Identify Patterns → Predict → Simulate → Learn from Outcomes
```

The prototype implements a stand-in for **Predict** and **Simulate** (deterministic, explainable formulas — see `docs/AI_DECISIONS.md`); **Identify Patterns** and **Learn from Outcomes** are not yet built (no feature store, no outcome-tracking loop).

## Target users

| Persona | Need |
|---|---|
| **Leadership** (org/product level) | Visualize branch, cluster, zone, and national health at a glance |
| **Zonal / Cluster / Branch Heads** | Catchment-level views, plus a non-spammy alerting mechanism for localized events (competitor activity, credit-risk shifts) |

The prototype now demonstrates **both** target personas (see `docs/USER_JOURNEY.md`), in two separate single-file builds:

- The **Zonal Head** view (`branch-pulse-view.html`) reflects Vikram's competitor-response scenario from the original design work — branch-operations first.
- The **Leadership** view (`leadership-view.html`) leans specifically into **market/product expansion-opportunity scanning**: which city, for which product, is the best next move. This is a deliberate *interpretation* of this table's "visualize branch, cluster, zone, and national health at a glance" line, not a literal 1:1 build of it — a more decision-forcing reading (pick a product → rank cities → compare → decide) rather than a passive health-rollup dashboard. Stated plainly here rather than dressed up as the original spec, per this project's honesty-over-marketing convention. Its scoring is a transparent, deterministic formula over hand-authored per-city inputs (`docs/AI_DECISIONS.md §5`).
