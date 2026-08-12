# For the CEO — `0.01` Define & maintain company type

**Finding companies by type works, but "Corporate" itself is mis-defined — it sweeps in banks and
advisories — and nobody can fix a wrong type. Four quick wins (≈ 10 dev-days) close the gaps.**

## What problem are we solving?

Every company must be classified Sponsor / PortCo / Corporate — the tag that drives targeting,
reporting and account ownership. Today it is set only by a data-provider import that computes
Corporate as simply "everything that is neither of the other two" — so banks, M&A advisories and
other firms we could never sell to are counted as prospects, and nobody can correct any of it by
hand.

## Where does it stand today?

Finding and filtering companies by type works end to end, fast and correct. **Maintaining** the
classification does not: there is no way to fix a wrong type, and refreshes overwrite silently.
Untyped-at-birth for product-created companies is accepted behaviour (2026-08-11).

**Reads work; the Corporate bucket over-counts by definition; maintenance fails silently in two
places.**

## What does this deliver?

Pass 1: the decision strip, this summary, and the journey health map with the confirmed scenario
list. Pass 2: the change briefs with every change tagged decided-or-CTO, and four junior-ready fix
cards. On demand: ops playbook, full journeys, surfaces and contract map.

## What will it cost?

Roughly **15–18 dev-days** total: quick wins ≈ 8–12 dev-days (reclassify service firms into the five
new types + backfill, make type correctable, consolidate the type vocabularies, audit bulk edits);
remaining ranked debt ≈ 4–6 dev-days. Refresh-protection and on-demand-refresh work (P2, P3) was
removed from scope by product on 2026-08-11; both pains stay on the register.

**Payback:** the quick wins make classification mean what the business means — and make it
correctable — for under a week of work.

## What we would fix

* **`QUICK WIN 1` · BLOCKER · `PRD GAP` — "Corporate" includes companies that can never be
  clients.** The tag is meant to mark possible Fintalent clients, but it is computed as merely "not
  a sponsor, not a portfolio company" — so every bank, M&A advisory and law firm imported from the
  provider lands in the prospect bucket, while the provider's own industry data on each record says
  exactly what they are. The fix: five new types — Advisory Firm, Law Firm, Banks, Accounting Firm,
  Other — driven by that industry data, leaving Corporate = possible clients only. **Cost of
  delay:** every Corporate-filtered target list and report keeps over-counting; SDR hours burn on
  firms that can never buy — against ~2–3 days of work once the mapping is signed (48h default).
* **`QUICK WIN 2` · BLOCKER — Nobody can fix a wrong company type.** Type drives targeting and
  reporting everywhere, yet no screen or control lets an admin set or correct it — and nothing flags
  the wrong ones, so bad classifications sit unnoticed. **Cost of delay:** every day, outreach and
  reports keep counting misclassified companies — wasted SDR effort and eroding trust in the data,
  against roughly 1–3 days of work.
* **`QUICK WIN 3` · CRITICAL — Three-plus competing notions of "company type" coexist:** the
  official three-way tag, a legacy free-text label (hidden from users when unclassified), automated
  classifications (visible only to beta testers), plus informal buckets and labels. **Cost of
  delay:** every cross-team report keeps reconciling mismatched numbers by hand, and decisions risk
  resting on the wrong ones.
* **`QUICK WIN 4` · MAJOR — Bulk edits leave no audit trail.** Individual edits are recorded and
  re-scored automatically; bulk edits re-score but skip the record of who changed what. **Cost of
  delay:** "who changed this?" stays unanswerable for bulk actions — an audit risk that compounds
  with every one.

**What already works well:** include/exclude type filtering with live counts, fast results — the
read side of this flow is in good shape. No spend needed here.

## Severity scoreboard

12 pain points, counted per class. Untyped-at-birth (P4) accepted as intended behaviour 2026-08-11
and removed.

| Class | Count |
|---|---|
| **BLOCKER** | 2 |
| **CRITICAL** | 2 |
| **MAJOR** | 5 |
| **MINOR** | 3 |

## The four quick wins

| # | | Worth |
|---|---|---|
| 1 · BLOCKER | **Redefine "Corporate" = possible client** — move banks, advisories and law firms into their own new types via the provider's industry data; backfill existing records | Every type-filtered list and report stops over-counting, for ~2–3 dev-days |
| 2 · BLOCKER | **Make company type correctable** — no screen or control lets an admin fix a wrong type today | Trustworthy target lists and reports, for ~1–3 dev-days |
| 3 · CRITICAL | **One canonical company type** — promote the official tag (eight values after win 1); retire the legacy free-text label (gate G4) | Cross-team reports finally agree, for ~3–5 dev-days |
| 4 · MAJOR | **Audit bulk edits** — bulk edits re-score but skip the record of who changed what | Every bulk action becomes traceable, for ~1–2 dev-days |

## Impact × effort — all 12 pain points

Higher = worse when it happens; further left = cheaper to fix. Quick-win zone = high impact, effort ≤ 2.

| ID | Pain | RPN |
|---|---|---|
| **P1** | No way to correct a company's type | 60 |
| **P13** | Corporate = catch-all; banks & advisories counted as clients · `PRD GAP` | 60 |
| **P3** | Refresh silently overwrites classification | 50 |
| **P5** | 3+ competing type taxonomies | 36 |
| **P6** | Bulk edits skip the audit trail | 30 |
| **P2** | On-demand refresh disconnected | 24 |
| **P9** | Label list has two sources of truth | 16 |
| **P10** | One-sided lifecycle validation | 16 |
| **P11** | Low-confidence classifications have no review queue | 16 |
| **P7** | Missing company shows an empty panel | 12 |
| **P8** | Bad filter values dropped without error | 10 |
| **P12** | Bulk "all matching" rejection offers no recovery guidance | 8 |

## What needs a CTO call

Open gates only. G1 accepted in rev 4 · G2 retired · G3 withdrawn (untyped-at-birth accepted).
`REVERSIBLE` defaults execute on silence; `ONE-WAY` gates wait for the change briefs.

| Gate | Type | Question | Recommendation |
|---|---|---|---|
| **G5** | `REVERSIBLE` | Adopt the five new service-firm types — Advisory Firm, Law Firm, Banks, Accounting Firm, Other — mapped from the provider's own industry data, leaving Corporate = possible clients only? | **Yes** — start with the draft mapping; the backfill is re-runnable, so an amended mapping re-applies cleanly. Executes on silence (48h) |
| **G4** | `ONE-WAY` | Taxonomy end-state: which vocabulary is canonical for "company type"? | Recommend promoting the official type tag and retiring the legacy text label — now gates quick-win card 3. **No silence default** — answer on the change-brief options |
