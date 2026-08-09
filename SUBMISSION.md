# Hackathon Submission

## Idea Title

**Branch Intelligence Simulator** — a hyperlocal, map-first intelligence platform for Piramal Finance.

Business, strategy, product, distribution, and branch-expansion teams at Piramal today make growth decisions off fragmented data — branch performance, demographics, demand, competition, geography, credit behavior — with no single hyperlocal view tying it together. This prototype unifies that into one live, drillable, simulate-able map, built for the two personas who actually need it:

- **Zonal Head** (`branch-pulse-view.html`) — a branch-operations view: zoom-driven Zone → Cluster → Branch drill-down over 733 real Piramal branch locations, health scoring, proactive alerts, and an AI Strategy Studio that turns "what happens if I move a slider" into "given a limited budget, what's the smartest intervention here."
- **Leadership** (`leadership-view.html`) — a market/product expansion-opportunity scanner: pick a product, see which of six candidate cities is the strongest launch/expansion opportunity, understand *why* in one sentence, and compare a shortlist side by side.

Both are single self-contained HTML files sharing one design system (Piramal's Mudra 2.0) and one live-map stack (Leaflet.js + OpenStreetMap — free, no signup, no API key). Full narrative detail: [`docs/IDEA.md`](./docs/IDEA.md) (the problem and original concept) and [`docs/USER_JOURNEY.md`](./docs/USER_JOURNEY.md) (both personas' walkthroughs, as actually built).

> **Status:** frontend prototype. Real branch location data; deterministic, transparent, explainable scoring/simulation logic standing in for a future trained model — never presented as real ML. See [`docs/AI_DECISIONS.md`](./docs/AI_DECISIONS.md) for the exact honesty line on every number in the app, and the Productionisation Plan below for what closing that gap actually takes.

---

## Productionisation Plan & Estimated Effort (Weeks)

Full detail and reasoning: [`PRODUCTION_PLAN.md`](./PRODUCTION_PLAN.md). Summary as of 2026-08-09, covering **both** persona views (they'd share one backend, one auth layer, one deployment):

| Dimension | State today |
|---|---|
| Frontend | Done — two polished, on-brand, functional single-file apps, real interaction models, no backend dependency |
| Data | **Real** branch locations (Zonal Head, 733/751 branches, data-quality-checked). **Synthetic** everything else on both views — health/KPI/simulation numbers and city-opportunity scores are deterministic formulas or hand-authored estimates, not real business metrics |
| Backend, AI/ML, Auth, Testing, Security, Compliance, Deployment | **None** on either view — see table below for what each takes |

| Workstream | Scope (both views) | Estimate |
|---|---|---|
| Backend & API | Data layer + CRUD for branches/zones/clusters *and* the city/product opportunity model; serve real (not embedded) data to both frontends | **5–7 weeks** |
| Real AI/ML pipeline | Feature store; trained demand/health models (replaces the Zonal Head's deterministic generator); trained market-opportunity scoring (replaces Leadership's hand-authored inputs); model registry, explainability | **8–12 weeks** — the long pole, depends on real historical data being available |
| Infrastructure | Hosting, CI/CD, environments, monitoring for one backend serving both frontends | **2–3 weeks** |
| Testing | Unit + integration + e2e across both views (currently zero automated tests) | **3–5 weeks** initial, then ongoing |
| Security | AuthN/AuthZ, RBAC (Zonal/Cluster/Branch Head vs. Leadership — a real requirement now that both roles exist with genuinely different data scopes) | **2–3 weeks** |
| Compliance | RBI data-localization review, PII classification, audit logging, legal sign-off | **3–5 weeks** — regulatory timeline, not engineering-bound |
| Deployment | Production hosting for both frontends, rollout plan, runbook | **1–2 weeks** |

### **Total estimated effort: ~24–37 weeks (6–9 months)** for a genuinely production-ready, two-persona system, assuming a small dedicated team and no reuse of existing Piramal infrastructure.

This shrinks meaningfully wherever backend/auth/hosting can plug into systems that already exist internally at Piramal — worth scoping explicitly before committing to a timeline. (If Leadership were deferred and only the Zonal Head view shipped, the estimate lands closer to **20–33 weeks**.)

**Immediate next steps**, in order: (1) get read access to real historical branch performance data and real market/demand/competitive data for candidate cities — this determines whether the AI/ML estimate above is realistic; (2) decide build-vs-integrate for backend/auth against existing Piramal platforms; (3) scope a minimum-viable real data source (even one) to replace the most-synthetic part of either view for the next demo; (4) loop in security/compliance early — BFSI review cycles, not engineering, are typically the true critical path. Full list: [`PRODUCTION_PLAN.md`](./PRODUCTION_PLAN.md#immediate-next-steps-if-this-moves-past-hackathon-stage).

---

## Setup Instructions

No install, no build, no backend, no API keys. Full detail: [`SETUP.md`](./SETUP.md).

**Requirements:** a modern browser; internet access (both views' live maps load Leaflet + OpenStreetMap from CDN, no signup/key required); Python 3 or any static file server.

**Run the Zonal Head view** (`branch-pulse-view.html`):
```bash
# Quick look — works immediately, offline map fallback:
open branch-pulse-view.html

# Full experience (live map, all 733 real branches) — serve it instead:
python3 -m http.server 8000
# then open http://localhost:8000/branch-pulse-view.html
```

**Run the Leadership view** (`leadership-view.html`) — **must** be served, no offline fallback:
```bash
python3 -m http.server 8000
# then open http://localhost:8000/leadership-view.html
```

**Deployment:** any static host works for both files (GitHub Pages, Netlify, Vercel, S3+CloudFront) — serve over `http(s)://`, not `file://`. No server-side logic to stand up.

---

*For everything beyond these three sections — full feature list, architecture, design decisions, AI/simulation honesty framing, demo script — see [`README.md`](./README.md) and the [`docs/`](./docs) folder.*
