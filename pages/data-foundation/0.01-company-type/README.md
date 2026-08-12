# `0.01` Define and maintain company type

**Activity** `0.01` — "Define and maintain company type"
**Stage** 0. Data Foundation · **Owner** short term SDR `R`, Senior Ops `S` · mid term SDR `S`, KAM `S`, Senior Ops `R`, Automation `S`
**Automation target** AI-assisted classification · *standardization layer for the CRM: PortCo, Sponsors, Corporates*
**Status** `Draft` · **Pass** `1 + 2` · **Last verified** 2026-08-10 on `develop` (both repos)
**Source** `working-docs/data-foundation/0.01 Company Type Flow Analysis.dc.html` (rev 5) + change briefs (rev 1) + backend/frontend detail (rev 1) · **Synced** 2026-08-12

# Define & maintain company type

**Reads work · Corporate over-tags non-clients · 2 silent failures · quick wins ≈ 10 days**

Sponsor / PortCo / Corporate classification on company records — plus the five service-firm types
added 2026-08-11 (Advisory Firm, Law Firm, Banks, Accounting Firm, Other) — how it is set, shown,
filtered, refreshed and (not) corrected.

**The 2026-08-10 product ruling:** Corporate = a Mergermarket-sourced company that could be a
Fintalent client. The code's catch-all derivation conflicts with it — flagged as a `PRD GAP` (P13),
with the code as today's reality.

| Derived | |
|---|---|
| **Flow** | Classify every company as Sponsor / PortCo / Corporate — service firms as Advisory Firm / Law Firm / Banks / Accounting Firm / Other — and keep that classification correct through data refreshes |
| **Entry points** | Admin companies list · company detail panel · Data Management (enrichment / health) |
| **Success condition** | The company record carries the correct type; every type-filtered list, count and downstream link reflects it |
| **Out of scope** | Dedupe/merge ([`0.02`](../0.02-clean-and-deduplicate-company-records/README.md)), contact hygiene (`0.05`), campaigns; legacy companies page noted, not traced |
| **Scenarios** | 7 — confirmed in rev 4 |

## Decide in 15 seconds

One question each, yes/no. You only veto. **Silence = the recommended default executes after 48h.**

| Who | The one question | Recommended default | If you do nothing (48h) |
|---|---|---|---|
| **CEO** | Approve ~15–18 dev-days to make Corporate mean "possible client", give service firms their own types, and make it all correctable? | **Yes** — start with the ~10-day quick-win bundle (reclassification · type editor · vocabulary consolidation · bulk audit); schedule the rest behind it | The quick-win bundle is scheduled; larger items queue for pass-2 briefs |
| **CPO** | Sign off the industry → type mapping for the five new types — Advisory Firm, Law Firm, Banks, Accounting Firm, Other (draft under gate G5)? | **Yes** — adopt the draft mapping; Corporate keeps only possible clients; the scenario list (7 after dropping S8) stays as confirmed in rev 4 | The draft mapping becomes the rule behind fix card 1's backfill — reversible; amend it any time |
| **COO** | Do the 2 symptom-check rows match what support hears, including "my Corporate list is full of banks"? | **Yes** — rows confirmed by looking; the known-error playbook stays on demand | Symptom rows stand as written; support can forward them as-is |
| **CTO** | Accept the G5 default (industry → type mapping from provider industry data), with G4 held for your change-brief answer? | **Yes** — G1 stands as accepted in rev 4 (G2 retired, G3 withdrawn — untyped-at-birth accepted); G5 is `REVERSIBLE` (re-runnable backfill) and safe to default | G5 executes with the default mapping; G4 stays open — `ONE-WAY`, never defaulted |
| **Tech lead** | Need the surfaces + contract map before pass 2 starts? | **Yes** — a full draft exists (rev 3 of the audit); it is refreshed and attached before pass 2 | The inventory ships together with the pass-2 briefs |

## Revision history

* **rev 5** — folds in the product ruling on the Corporate definition (new P13).
* **2026-08-11** — refresh-protection (P3) and on-demand-refresh (P2) removed from scope on product
  direction; five new company types added; untyped-at-birth (P4) accepted as intended behaviour.

## Your page

| You are | Read |
|---|---|
| CEO | [For the CEO](for-the-ceo.md) |
| COO / Operations | [For operations](for-operations.md) |
| CTO | [For the CTO](for-the-cto.md) |
| Developers | [For developers](for-developers.md) |
| Tech team | [For the tech team](for-the-tech-team.md) |

Ops playbook and the full journeys / pain register are on demand.
