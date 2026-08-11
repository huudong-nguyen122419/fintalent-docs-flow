# 0.01 · Define and maintain company type

> **0. Data Foundation** · **Lean flow audit · pass 1 + 2 · rev 5**

{% hint style="warning" %}
**Verdict:** reads work · Corporate over-tags non-clients · 2 silent failures · quick wins ≈ 10 days
{% endhint %}

Sponsor / PortCo / Corporate classification on company records — plus the five service-firm types added 2026-08-11 (Advisory Firm, Law Firm, Banks, Accounting Firm, Other) — how it is set, shown, filtered, refreshed and (not) corrected. Derived from the connected repos (branch `develop`, read 2026-08-10); dispute any derived value below and it is corrected before pass 2.

Rev 5 adds the product ruling of 2026-08-10 — **Corporate = a Mergermarket-sourced company that could be a Fintalent client**; the code's catch-all derivation conflicts with it, flagged as a PRD gap (P13) with the code as today's reality.

## On this flow

<table><thead><tr><th width="227">Section</th><th width="168">For</th><th>What it holds</th></tr></thead><tbody><tr><td><a href="who-does-it-and-scope.md">Who does it &#x26; scope</a></td><td>—</td><td>Owner today and in the mid-term model; what the flow covers, where it starts, what counts as success.</td></tr><tr><td><a href="decision-strip.md">0 · Decision strip</a></td><td>all stakeholders</td><td>One yes/no question each. Silence executes the default after 48h.</td></tr><tr><td><a href="executive-summary.md">1 · Executive summary</a></td><td>CEO</td><td>The problem, the five findings, quick wins, severity and the impact grid.</td></tr><tr><td><a href="journey-health-map.md">2 · Journey health map</a></td><td>COO + CPO</td><td>The seven steps as a person walks them, plus scenarios and the symptom check.</td></tr><tr><td><a href="change-briefs.md">3 · Change briefs</a></td><td>CTO</td><td>Backend and frontend contracts, decision gates, completion checklists.</td></tr><tr><td><a href="build-instructions.md">9 · Build instructions</a></td><td>junior/mid devs</td><td>Four fix cards with exact file paths and Gherkin acceptance.</td></tr></tbody></table>
