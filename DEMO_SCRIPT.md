# Demo Script

A ~5–7 minute walkthrough. Each beat: talking point, business value, user story, expected interaction.

---

### 1. The hook

**Talking point:** "Piramal's branch, product, and strategy teams today work off fragmented data — performance, demographics, competition, credit behavior — with no single hyperlocal view. This is what a unified view looks like."
**Business value:** Frames every feature that follows as solving a named, real problem — not a tech demo for its own sake.
**User story:** Every persona in `docs/IDEA.md`.
**Interaction:** Open the app cold. Let the national zone-circle view load.

---

### 2. National view — Zones as circles

**Talking point:** "Five zones, color-coded by health, sized by branch count. This is Leadership's view — the whole network at a glance."
**Business value:** One artifact, every level of the org.
**User story:** Leadership persona.
**Interaction:** Point out the 5 zone circles and their health-score coloring. Click the "Southern" zone chip to jump there — mention it's 328 real branches, not a mockup.

---

### 3. Zoom in — Clusters as blocks

**Talking point:** "Zoom in, and zones resolve into clusters — geographic groupings of nearby branches. Same map, same interaction, more detail."
**Business value:** Demonstrates the Zone → Cluster → Branch org hierarchy is a first-class concept, not just a filter.
**User story:** Cluster Head persona.
**Interaction:** Scroll to zoom into a cluster (e.g., Pune Cluster). Point out the shape change (circle → block) as an intentional legibility choice.

---

### 4. Zoom in further — Branches as pins, and the alert

**Talking point:** "Now individual branches. This one — Pune, Hadapsar — has an active alert: health is projected to slip 6 points next week."
**Business value:** This is the proactive-alerting pitch — teams currently find out about problems at quarterly review, not before they happen.
**User story:** Zonal Head (Vikram) persona — see `docs/USER_JOURNEY.md` for the full narrative.
**Interaction:** Zoom/click into Pune's pin. Click "View reason" on the alert banner — read the specific driver (RM attrition + competitor groundbreaking).

---

### 5. Health, KPIs, and the insight

**Talking point:** "Health score, four sub-scores, three trending KPIs — and an insight: retail demand is up 18% in this catchment, a local 'lipstick effect' signal, right before a competitor opens nearby."
**Business value:** Demonstrates the "Identify Patterns" stage — turning raw signals into a narrative a non-analyst can act on.
**User story:** Zonal Head.
**Interaction:** Scroll the right panel; expand the insight's "Why?" disclosure to show supporting drivers.

---

### 6. Strategy Studio — the centerpiece

**Talking point:** "Now the flagship. Instead of asking 'what happens if I move a slider,' this asks the real question: 'given a limited budget, what is the smartest intervention here?' Watch — the AI already has an answer."
**Business value:** This is the "Simulate" stage of the core AI flow reframed as an executive planning workspace — prescriptive, comparative, defensible. Most competitor dashboards stop at describing; this recommends, quantifies, and lets you stress-test before spending a rupee.
**User story:** Zonal Head, in the exact moment the original product brief describes — but now with a decision workspace, not a form.
**Interaction:** Click "Simulate →". The screen expands into the full-screen Studio — **call out that the map fills the surface and stays live** (no modal covering it). Read the **hero recommendation** aloud (strategy, ROI, confidence, investment, payback, risk). Then drag the **budget slider** and flip **deployment** from Pilot to Zone — narrate the nine KPIs animating and the **coverage radius growing on the map with branches lighting up**. Point to the **3-way trade-off** and the "Best ROI / Lowest Risk" badges: "the tool doesn't just answer — it shows the alternatives it rejected and why." Expand **AI reasoning** for the one-line justifications.

---

### 7. Decide — approve & business case

**Talking point:** "Everything a review committee needs is assembled here — expected revenue, ROI, payback, top risks, key assumptions. One tap approves it; or export the business case to take into the room."
**Business value:** Closes the loop from signal → recommendation → stress-test → defensible decision, all on one screen.
**User story:** Zonal Head.
**Interaction:** Scroll to the executive summary. Click **Approve rollout** → show the toast + the session-persistent "Recently actioned" entry on the dashboard. Optionally click **Export** to trigger the downloadable business case. Mention Save scenario / Compare later for multi-option planning.

---

### 8. Zoom back out — the leaderboard

**Talking point:** "Zoom back out, and the same panel becomes a leaderboard — best and worst performing branches in this cluster, one click from any of them back into full detail."
**Business value:** Same screen serves monitoring (leadership) and action (branch-level) without a mode switch.
**User story:** Leadership / Cluster Head.
**Interaction:** Pan/zoom back out; click a "Needs attention" leaderboard row to jump straight into that branch.

---

### 8b. Leadership view — product-expansion opportunity scanner (companion view)

**Talking point:** "That was the Zonal Head. For the Leadership / product-strategy persona, the same platform answers a different question — not 'is this branch okay?' but 'which product should we launch or expand into which city next?' Here's that view." Open `leadership-view.html`. "Six candidate cities on a live map, each coloured by how strong an opportunity it is *for the selected product* — and the single biggest opportunity pulses."
**Business value:** Demonstrates the second target persona from the brief, and reframes "visualize national health" as a concrete, decision-forcing expansion call — the more compelling read for a strategy leader.
**User story:** Leadership persona (Anjali, Head of Product Strategy) — see `docs/USER_JOURNEY.md`.
**Interaction:** Flip the top-right product selector Gold Loan → Personal Loan and call out the pins recolouring and the ranked list re-sorting — the biggest opportunity *moves* (Jaipur → Indore). Click a city pin: read its score, band, and the one-line insight that names *why* (e.g. "underlying demand is the strongest signal at 79/100"). Add two cities to compare and open the comparison modal — "the same six cities, scored independently per product, laid side by side." Close with the honesty beat: the scoring formula is real and transparent, but the per-city inputs are hand-authored estimates (`docs/AI_DECISIONS.md §5`) — same convention as the rest of this prototype.

---

### 9. (Optional, strong for technical judges) The data-quality story

**Talking point:** "This map uses 733 real Piramal branch locations, not synthetic pins. While integrating them, we actually caught a real data error — one branch's coordinate had a dropped decimal point, which was dragging an entire zone's map position into Chad. We caught it, verified it against real-world coordinates rather than trusting a blind statistical rule, and fixed it — that fix is in `CHANGELOG.md`."
**Business value:** Signals engineering rigor and honesty — the kind of story that differentiates a team that actually shipped something real from one that faked a screenshot.
**User story:** N/A — this is a credibility beat for judges, not a user-facing feature.
**Interaction:** None required — a talking point, optionally backed by opening `CHANGELOG.md`.

---

### 10. The close — what this scales to

**Talking point:** "This is a working slice of a 6-layer production architecture we designed up front — real ingestion, geospatial indexing, an AI feature store, a predictive/simulation core, alerting, and a continuous learning loop. We can walk through exactly what's real today versus what's next."
**Business value:** Positions the demo as a credible first step, not a finished claim — and shows a team that thought about production from day one.
**User story:** N/A — judging-panel close.
**Interaction:** Show `architecture_flowchart.png` and/or `PRODUCTION_PLAN.md`'s maturity table.
