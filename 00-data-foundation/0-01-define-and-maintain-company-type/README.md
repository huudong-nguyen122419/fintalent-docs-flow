# 0.01 · Define and maintain company type

> **0. Data Foundation** · **Lean flow audit · pass 1 + 2 · rev 5**

> **Verdict:** reads work · Corporate over-tags non-clients · 2 silent failures · quick wins ≈ 10 days

Sponsor / PortCo / Corporate classification on company records — plus the five service-firm types added 2026-08-11 (Advisory Firm, Law Firm, Banks, Accounting Firm, Other) — how it is set, shown, filtered, refreshed and (not) corrected. Derived from the connected repos (branch `develop`, read 2026-08-10); dispute any derived value below and it is corrected before pass 2.

Rev 5 adds the product ruling of 2026-08-10 — **Corporate = a Mergermarket-sourced company that could be a Fintalent client**; the code's catch-all derivation conflicts with it, flagged as a PRD gap (P13) with the code as today's reality.

## On this flow

| Section | For | What it holds |
|---|---|---|
| [Who does it](who-does-it.md) | — | Owner today and in the mid-term model. |
| [Scope](scope.md) | — | What the flow covers, where it starts, what counts as success. |
| [0 · Decision strip](decision-strip.md) | all stakeholders | One yes/no question each. Silence executes the default after 48h. |
| [1 · Executive summary](executive-summary.md) | CEO | The problem, the five findings, quick wins, severity and the impact grid. |
| [2 · Journey health map](journey-health-map.md) | COO + CPO | The seven steps as a person walks them, plus scenarios and the symptom check. |
| [3 · Change briefs](change-briefs.md) | CTO | Backend and frontend contracts, decision gates, completion checklists. |
| [9 · Build instructions](build-instructions.md) | junior/mid devs | Four fix cards with exact file paths and Gherkin acceptance. |
