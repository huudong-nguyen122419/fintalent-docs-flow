# 1 · Executive summary

> part of [0.01 · Define & maintain company type](./)

**For: CEO.** _Business language, zero code terms, zero raw scores in prose._

{% hint style="danger" %}
**Takeaway:** finding companies by type works, but **"Corporate" itself is mis-defined** — it sweeps in banks and advisories — and **nobody can fix a wrong type**; four quick wins (≈ 10 dev-days) close the gaps.
{% endhint %}

## <mark style="color:blue;">The problem</mark>

Every company must be classified Sponsor / PortCo / Corporate — the tag that drives targeting, reporting and account ownership. Today it is set **only** by a data-provider import that computes Corporate as simply _"everything that is neither of the other two"_ — so banks, M\&A advisories and other firms we could never sell to are counted as prospects, and nobody can correct any of it by hand.

## <mark style="color:blue;">Where it stands today</mark>

Finding and filtering companies by type works end-to-end, fast and correct. **Maintaining** the classification does not: there is no way to fix a wrong type and refreshes overwrite silently. Untyped-at-birth for product-created companies is accepted behaviour (2026-08-11).

**Status: reads work; the Corporate bucket over-counts by definition; maintenance fails silently in two places.**

## <mark style="color:blue;">What this document delivers</mark>

**Pass 1:** the decision strip, this summary, and the journey health map with the scenario list (confirmed — explicit go received). **Pass 2** (included below): backend + frontend change briefs ([§3](change-briefs.md)) with every change tagged decided-or-CTO, and four junior-ready fix cards ([§9](build-instructions.md)). **On demand:** ops playbook, full journeys, technical appendix (drafted in rev 3).

## <mark style="color:blue;">What it costs</mark>

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

|        | E1 · <0.5 day                                                                                                                       | E2                                                                                                                                                                       | E3 · 1–3 days                               | E4                                        | E5 · >5 days |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------- | ----------------------------------------- | ------------ |
| **S5** |                      　                                                                                                              | <mark style="color:$danger;background-color:$success;">**P1**</mark>**&#x20;&#x20;**<mark style="color:orange;background-color:$success;">**P3**</mark>                  |                                             |                                           |              |
| **S4** | <mark style="color:$warning;background-color:$success;">**P2**</mark>                                                               | <mark style="color:$danger;background-color:$success;">**P13**</mark>                                                                                                    |                                             |                                           |              |
| **S3** |                                                                                                                                     | <mark style="color:$warning;">**P6**</mark>                                                                                                                              |                                             | <mark style="color:orange;">**P5**</mark> |              |
| **S2** | <mark style="color:$info;">**P8**</mark> · <mark style="color:$warning;">**P10**</mark> · <mark style="color:$info;">**P12**</mark> | <mark style="color:$info;">**P7**</mark> · <mark style="color:$warning;">**P11**</mark>                                                                                  | <mark style="color:$warning;">**P9**</mark> |                                           |              |
| **S1** |                                                                                                                                     |                                                                                                                                                                          |                                             |                                           |              |

## Pain register

<table><thead><tr><th width="131"></th><th width="468">Pain</th><th align="right">RPN</th></tr></thead><tbody><tr><td><mark style="color:$danger;background-color:$danger;"><strong>P1</strong></mark></td><td>No way to correct a company's type</td><td align="right"><strong>60</strong></td></tr><tr><td><mark style="color:$danger;background-color:$danger;"><strong>P13</strong></mark></td><td>Corporate = catch-all; banks &#x26; advisories counted as clients · <mark style="color:red;"><strong>PRD GAP</strong></mark></td><td align="right"><strong>60</strong></td></tr><tr><td><mark style="color:orange;background-color:orange;"><strong>P3</strong></mark></td><td>Refresh silently overwrites classification</td><td align="right">50</td></tr><tr><td><mark style="color:orange;background-color:orange;"><strong>P5</strong></mark></td><td>3+ competing type taxonomies</td><td align="right">36</td></tr><tr><td><mark style="color:$warning;background-color:$warning;"><strong>P6</strong></mark></td><td>Bulk edits skip the audit trail</td><td align="right">30</td></tr><tr><td><mark style="color:$warning;background-color:$warning;"><strong>P2</strong></mark></td><td>On-demand refresh disconnected</td><td align="right">24</td></tr><tr><td><mark style="color:$warning;background-color:$warning;"><strong>P9</strong></mark></td><td>Label list has two sources of truth</td><td align="right">16</td></tr><tr><td><mark style="color:$warning;background-color:$warning;"><strong>P10</strong></mark></td><td>One-sided lifecycle validation</td><td align="right">16</td></tr><tr><td><mark style="color:$warning;background-color:$warning;"><strong>P11</strong></mark></td><td>Low-confidence classifications have no review queue</td><td align="right">16</td></tr><tr><td><mark style="color:$info;background-color:$info;"><strong>P7</strong></mark></td><td>Missing company shows an empty panel</td><td align="right">12</td></tr><tr><td><mark style="color:$info;background-color:$info;"><strong>P8</strong></mark></td><td>Bad filter values dropped without error</td><td align="right">10</td></tr><tr><td><mark style="color:$info;background-color:$info;"><strong>P12</strong></mark></td><td>Bulk "all matching" rejection offers no recovery guidance</td><td align="right">8</td></tr></tbody></table>

## Open gates

_Open gates only (G1 accepted in rev 4 · G2 retired · G3 withdrawn — untyped-at-birth accepted). REVERSIBLE defaults execute on silence; ONE-WAY gates wait for_ [_§3_](change-briefs.md)_._

<table><thead><tr><th width="170">Gate</th><th>Question</th><th>Auditor recommendation</th></tr></thead><tbody><tr><td><mark style="color:blue;"><strong>G5</strong></mark><br><mark style="color:$success;background-color:$success;">REVERSIBLE</mark></td><td>Adopt the five new service-firm types — Advisory Firm, Law Firm, Banks, Accounting Firm, Other — mapped from the provider's own industry data, leaving Corporate = possible clients only?</td><td><mark style="color:$success;"><strong>Yes</strong></mark> — start with the draft mapping (gate G5, <a href="change-briefs.md">§3</a>); the backfill is re-runnable, so an amended mapping re-applies cleanly. <mark style="color:red;"><strong>Executes on silence (48h)</strong></mark><strong>.</strong></td></tr><tr><td><mark style="color:blue;"><strong>G4</strong></mark><br><mark style="color:$danger;background-color:$danger;">ONE-WAY</mark></td><td>Taxonomy end-state: which vocabulary is canonical for "company type"?</td><td>Promote the official type tag and retire the legacy text label — now gates quick-win card 3. <strong>No silence default</strong> — answer on the <a href="change-briefs.md">§3</a> options.</td></tr></tbody></table>
