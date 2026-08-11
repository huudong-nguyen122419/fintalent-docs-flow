# 1 · Executive summary

> part of [0.01 · Define & maintain company type](./)

**For: CEO.** _Business language, zero code terms, zero raw scores in prose._

{% hint style="danger" %}
**Takeaway:** finding companies by type works, but **"Corporate" itself is mis-defined** — it sweeps in banks and advisories — and **nobody can fix a wrong type**; four quick wins (≈ 10 dev-days) close the gaps.
{% endhint %}

## The problem

Every company must be classified Sponsor / PortCo / Corporate — the tag that drives targeting, reporting and account ownership. Today it is set **only** by a data-provider import that computes Corporate as simply _"everything that is neither of the other two"_ — so banks, M\&A advisories and other firms we could never sell to are counted as prospects, and nobody can correct any of it by hand.

## Where it stands today

Finding and filtering companies by type works end-to-end, fast and correct. **Maintaining** the classification does not: there is no way to fix a wrong type and refreshes overwrite silently. Untyped-at-birth for product-created companies is accepted behaviour (2026-08-11).

**Status: reads work; the Corporate bucket over-counts by definition; maintenance fails silently in two places.**

## What this document delivers

**Pass 1:** the decision strip, this summary, and the journey health map with the scenario list (confirmed — explicit go received). **Pass 2** (included below): backend + frontend change briefs ([§3](change-briefs.md)) with every change tagged decided-or-CTO, and four junior-ready fix cards ([§9](build-instructions.md)). **On demand:** ops playbook, full journeys, technical appendix (drafted in rev 3).

## What it costs

Roughly **15–18 dev-days total**: quick wins ≈ 8–12 dev-days (reclassify service firms into the five new types + backfill, make type correctable, consolidate the type vocabularies, audit bulk edits); the remaining ranked debt ≈ 4–6 dev-days. Refresh-protection and on-demand-refresh work (P2, P3) was removed from scope by product on 2026-08-11; both pains stay on the register.

**Payback:** the quick wins make classification mean what the business means — and make it correctable — for under a week of work.

## The five findings

**1 ·&#x20;**<mark style="color:$success;">**`QUICK WIN 1`**</mark> <mark style="color:$success;">**`BLOCKER`**</mark> <mark style="color:$success;">**`PRD GAP`**</mark>**&#x20;— "Corporate" includes companies that can never be clients**

The tag is meant to mark possible Fintalent clients, but it is computed as merely _"not a sponsor, not a portfolio company"_ — so every bank, M\&A advisory and law firm imported from the provider lands in the prospect bucket, **while the provider's own industry data on each record says exactly what they are**. The fix: five new types — Advisory Firm, Law Firm, Banks, Accounting Firm, Other — driven by that industry data, leaving Corporate = possible clients only.

_Cost of delay:_ every Corporate-filtered target list and report keeps over-counting; SDR hours burn on firms that can never buy — against \~2–3 days of work once the industry → type mapping is signed (48h default).

**2 ·&#x20;**<mark style="color:$success;">**`QUICK WIN 2`**</mark> <mark style="color:$success;">**`BLOCKER`**</mark>**&#x20;— Nobody can fix a wrong company type**

Type drives targeting and reporting everywhere, yet no screen or control lets an admin set or correct it — and nothing flags the wrong ones, so bad classifications sit unnoticed.

_Cost of delay:_ every day, outreach and reports keep counting misclassified companies — wasted SDR effort and eroding trust in the data, against roughly 1–3 days of work.

**3 ·&#x20;**<mark style="color:$success;">**`QUICK WIN 3`**</mark> <mark style="color:$success;">**`CRITICAL`**</mark>**&#x20;— Three-plus competing notions of "company type" coexist**

The official three-way tag, a legacy free-text label (hidden from users when unclassified), automated classifications (visible only to beta testers), plus informal buckets and labels.

_Cost of delay:_ every cross-team report keeps reconciling mismatched numbers by hand, and decisions risk resting on the wrong ones.

**4 ·&#x20;**<mark style="color:$success;">**`QUICK WIN 4`**</mark> <mark style="color:$success;">**`MAJOR`**</mark>**&#x20;— Bulk edits leave no audit trail**

Individual edits are recorded and re-scored automatically; bulk edits re-score but skip the record of who changed what.

_Cost of delay:_ _"who changed this?"_ stays unanswerable for bulk actions — an audit risk that compounds with every one.

**5 · Finding companies by type works well**

Include/exclude type filtering with live counts, fast results — the read side of this flow is in good shape. _No spend needed here._

## Quick wins

_Four fixes, ≈ 8–12 dev-days. Build instructions in_ [_§9_](build-instructions.md)_. The eight pains outside this bundle stay on the register below._

<table><thead><tr><th width="139"></th><th width="327">Fix</th><th>Worth</th></tr></thead><tbody><tr><td><mark style="color:$success;"><strong><code>QUICK WIN 1</code></strong></mark><br><mark style="color:$danger;background-color:$danger;"><strong>BLOCKER</strong></mark></td><td><strong>Redefine "Corporate" = possible client.</strong> Move banks, advisories and law firms into their own new types — Advisory Firm / Law Firm / Banks / Other — via the provider's industry data; backfill existing records.</td><td>every type-filtered list and report stops over-counting, for ~2–3 dev-days</td></tr><tr><td><mark style="color:$success;"><strong><code>QUICK WIN 2</code></strong></mark><br><mark style="color:$danger;background-color:$danger;"><strong>BLOCKER</strong></mark></td><td><strong>Make company type correctable.</strong> No screen or control lets an admin fix a wrong type today.</td><td>trustworthy target lists and reports, for ~1–3 dev-days</td></tr><tr><td><mark style="color:$success;"><strong><code>QUICK WIN 3</code></strong></mark><br><mark style="color:orange;background-color:orange;"><strong>CRITICAL</strong></mark></td><td><strong>One canonical company type.</strong> Promote the official tag (eight values after win 1); retire the legacy free-text label — gate G4.</td><td>cross-team reports finally agree, for ~3–5 dev-days</td></tr><tr><td><mark style="color:$success;"><strong><code>QUICK WIN 4</code></strong></mark><br><mark style="color:$warning;background-color:$warning;"><strong>MAJOR</strong></mark></td><td><strong>Audit bulk edits.</strong> Bulk edits re-score but skip the record of who changed what.</td><td>every bulk action becomes traceable, for ~1–2 dev-days</td></tr></tbody></table>

## Severity scoreboard

| <mark style="color:$danger;background-color:$danger;">BLOCKER</mark> | <mark style="color:orange;background-color:orange;">CRITICAL</mark> | <mark style="color:$warning;background-color:$warning;">MAJOR</mark> | <mark style="color:$info;background-color:$info;">MINOR</mark> |
| :------------------------------------------------------------------: | :-----------------------------------------------------------------: | :------------------------------------------------------------------: | :------------------------------------------------------------: |
|                                 **2**                                |                                **2**                                |                                 **5**                                |                              **3**                             |

_12 pain points found, counted per class. Untyped-at-birth (P4) accepted as intended behaviour 2026-08-11 and removed._

## Impact × effort

_Rows = how bad it is when it happens (S5 worst) · columns = how expensive it is to fix (E1 cheapest). Text colour is the severity class; a green cell sits in the quick-win zone._

|        | E1 · <0.5 day | E2 | E3 · 1–3 days | E4 | E5 · >5 days |
| ------ | ------------- | -- | ------------- | -- | ------------ |
| **S5** | <mark style="background-color:$success;">　</mark> | <mark style="color:$danger;background-color:$success;">**P1**</mark> · <mark style="color:orange;background-color:$success;">**P3**</mark> | | | |
| **S4** | <mark style="color:$warning;background-color:$success;">**P2**</mark> | <mark style="color:$danger;background-color:$success;">**P13**</mark> | | | |
| **S3** | | <mark style="color:$warning;">**P6**</mark> | | <mark style="color:orange;">**P5**</mark> | |
| **S2** | <mark style="color:$info;">**P8**</mark> · <mark style="color:$warning;">**P10**</mark> · <mark style="color:$info;">**P12**</mark> | <mark style="color:$info;">**P7**</mark> · <mark style="color:$warning;">**P11**</mark> | <mark style="color:$warning;">**P9**</mark> | | |
| **S1** | | | | | |

**The green zone** is the top-left corner — hurts a lot, costs little. Anything landing there is quick-win territory by default, and four pains sit in it: **P1, P3, P2, P13**.

**But the quick-win bundle is P13, P1, P5, P6 — not the green four.** Two moved out and two moved in, for reasons the grid cannot see:

| Pain       | Movement                                                                  | Why                                                                                                                     |
| ---------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **P3, P2** | <mark style="background-color:$success;">green zone</mark> → out of scope | descoped by product on 2026-08-11 — a scoring grid does not outrank a product decision; both stay on the register below |
| **P5**     | S3 / E4 → <mark style="color:orange;">quick win 3</mark>                  | the priciest fix still on the board, promoted because every other taxonomy fix waits behind it                          |
| **P6**     | S3 / E2 → <mark style="color:$warning;">quick win 4</mark>                | audit gaps compound — cheap now, unrecoverable later                                                                    |

_Read the grid as the opening argument, not the verdict._

## Pain register

|                                                                        | Pain                                                                       |    RPN |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------- | -----: |
| <mark style="color:$danger;background-color:$danger;">**P1**</mark>    | No way to correct a company's type                                         | **60** |
| <mark style="color:$danger;background-color:$danger;">**P13**</mark>   | Corporate = catch-all; banks & advisories counted as clients · **PRD GAP** | **60** |
| <mark style="color:orange;background-color:orange;">**P3**</mark>      | Refresh silently overwrites classification                                 |     50 |
| <mark style="color:orange;background-color:orange;">**P5**</mark>      | 3+ competing type taxonomies                                               |     36 |
| <mark style="color:$warning;background-color:$warning;">**P6**</mark>  | Bulk edits skip the audit trail                                            |     30 |
| <mark style="color:$warning;background-color:$warning;">**P2**</mark>  | On-demand refresh disconnected                                             |     24 |
| <mark style="color:$warning;background-color:$warning;">**P9**</mark>  | Label list has two sources of truth                                        |     16 |
| <mark style="color:$warning;background-color:$warning;">**P10**</mark> | One-sided lifecycle validation                                             |     16 |
| <mark style="color:$warning;background-color:$warning;">**P11**</mark> | Low-confidence classifications have no review queue                        |     16 |
| <mark style="color:$info;background-color:$info;">**P7**</mark>        | Missing company shows an empty panel                                       |     12 |
| <mark style="color:$info;background-color:$info;">**P8**</mark>        | Bad filter values dropped without error                                    |     10 |
| <mark style="color:$info;background-color:$info;">**P12**</mark>       | Bulk "all matching" rejection offers no recovery guidance                  |      8 |

## Open gates

_Open gates only (G1 accepted in rev 4 · G2 retired · G3 withdrawn — untyped-at-birth accepted). REVERSIBLE defaults execute on silence; ONE-WAY gates wait for_ [_§3_](change-briefs.md)_._

| Gate                                                                             | Question                                                                                                                                                                                  | Auditor recommendation                                                                                                                                                          |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **G5** <mark style="color:$success;background-color:$success;">REVERSIBLE</mark> | Adopt the five new service-firm types — Advisory Firm, Law Firm, Banks, Accounting Firm, Other — mapped from the provider's own industry data, leaving Corporate = possible clients only? | **Yes** — start with the draft mapping (gate G5, [§3](change-briefs.md)); the backfill is re-runnable, so an amended mapping re-applies cleanly. **Executes on silence (48h).** |
| **G4** <mark style="color:$danger;background-color:$danger;">ONE-WAY</mark>      | Taxonomy end-state: which vocabulary is canonical for "company type"?                                                                                                                     | Promote the official type tag and retire the legacy text label — now gates quick-win card 3. **No silence default** — answer on the [§3](change-briefs.md) options.             |
