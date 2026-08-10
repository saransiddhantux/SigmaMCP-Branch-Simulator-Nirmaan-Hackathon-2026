# Future Scope

Tied to `PRODUCTION_PLAN.md`'s maturity assessment — this is the roadmap, that document is the honest cost estimate.

## Near-term (makes the prototype more credible, not yet "production")

- **Real historical performance data**, even for a handful of branches, to replace the deterministic scorecard generator's most visible gaps (`docs/AI_DECISIONS.md`).
- **Official cluster boundaries**, if/when available, replacing the k-means approximation.
- **Offline map parity** — either a real lat/lon-aware map library for the no-internet fallback, or an honest "requires internet" gate instead of a silently smaller experience (see `SETUP.md` Known Issues).
- **District/sub-cluster drill-down** — the current 3-tier hierarchy (Zone/Cluster/Branch) could extend one level deeper using the district field already present in the source data.
- **A real product catalog and action-outcome tracking** — even a simple spreadsheet-backed log of "which simulated actions were actually approved and what happened" starts building the data needed for the Continuous Learning Loop below.

## Long-term (the full vision — see `architecture_flowchart.png`)

Mapped to the original 6-layer target architecture:

1. **Data Ingestion & Integration** — real internal transactional/credit data pipelines + external geospatial/economic feeds (currently: none; the prototype only has branch location data)
2. **Geospatial Indexing** — the originally conceived ~3km H3 hexagon grid, replacing/complementing the current Zone/Cluster/Branch hierarchy for true hyperlocal (not just organizational) granularity
3. **AI Feature Store & Pattern Engine** — real feature aggregation + a look-alike similarity engine for expansion recommendations (currently: a fixed insight-template pool)
4. **Predictive & Simulation Core** — real trained models (XGBoost/LightGBM per the architecture doc) replacing every deterministic formula in `docs/AI_DECISIONS.md`
5. **User-Serving & Alerting API** — a real backend serving this UI, plus the originally scoped mobile push-notification engine (non-spam, interval-based) — entirely unbuilt today
6. **Continuous Learning Loop** — comparing actual outcomes against past simulations to detect model drift and retrain — requires #4 and #5 to exist first

## Sequencing note

`PRODUCTION_PLAN.md` estimates 5–8 months for the full picture above. The single highest-leverage next step is **Layer 1 (real data)** — every other layer's credibility depends on it, and it's the one piece no amount of good frontend engineering can substitute for.
