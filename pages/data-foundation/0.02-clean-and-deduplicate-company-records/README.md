# `0.02` Clean and deduplicate company records

**Activity** `0.02` — "Clean and deduplicate company records"
**Stage** 0. Data Foundation · **Owner** short term SDR `R` · mid term SDR `S`, Automation `R`
**Automation target** Automated deduplication and enrichment
**Status** `Draft` · **Pass** `1` · **Last verified** 2026-08-11 on `develop` (both repos)
**Source** `working-docs/data-foundation/0.02 Company Dedupe Flow Analysis.dc.html` (rev 1) · **Synced** 2026-08-12

# Remove duplicates & correct company data

**Merge works · 1 contract gap · 2 silent risks · quick wins ≈ 2 days**

How duplicate companies are found, reviewed and merged, and how incomplete, inconsistent or
outdated records get corrected — traced end to end from the admin screens through the merge and
health services to the database. **Silence = approval** on every derived value and every
`REVERSIBLE` default.

| Derived | |
|---|---|
| **Flow** | Find records that describe the same real company, merge them into one survivor (people, lists and campaign links intact), and keep every record complete and current |
| **Entry points** | "Suspected duplicate" warning chip on any company row · bulk-select → Merge in the companies list · Data Management → Company health dashboard (drill) · per-row "Refresh from provider" button |
| **Success condition** | One surviving record per real company; nothing attached to a merged-away record is lost; warnings reflect reality; stale records refresh on schedule |
| **Out of scope** | Company type rules ([`0.01`](../0.01-company-type/README.md)), contact dedupe, AI enrichment autofixes, the Group/affiliate merge mode (traced, healthy, not this flow's debt) |
| **Scenarios** | 8 — confirmed by default; a CPO veto trims the list |

## Decide in 15 seconds

One question each, yes/no. You only veto. **Silence = the recommended default executes after 48h.**
`ONE-WAY` items are never defaulted.

| Who | The one question | Recommended default | If you do nothing (48h) |
|---|---|---|---|
| **CEO** | Approve ≈ 2 dev-days of quick wins now (fix the after-merge confusion + make the nightly upkeep provably on), with the bigger items decided in pass 2? | **Yes** — the merge tool itself is well built; the debt is around it and the first two fixes are cheap | Fix cards 1–2 are scheduled; pass-2 briefs follow for the rest |
| **CPO** | Confirm the 8 scenarios, and that "remove duplicates" is meant to include same-name pairs the scanner can't currently see (P3)? | **Yes** — scenarios stand; P3 gets a cheap measurement first (count the invisible pairs) before any detection work is bought | Scenario list is confirmed as written; the P3 measurement runs with pass 2 |
| **COO** | Do the 3 symptom-check rows match what support and the data team actually hear? | **Yes** — confirm by looking, not memory; the known-error playbook stays on demand | Symptom rows stand as written; support can forward them as-is |
| **CTO** | Accept the two `REVERSIBLE` gate defaults (G1 refresh-after-merge, G3 upkeep alert), with G2 — the people-link backfill — held `ONE-WAY` for your pass-2 answer? | **Yes** — G1 and G3 are additive and safe to default; G2 rewrites many people records and waits for the change briefs plus the verification count in fix card 3 | G1 + G3 execute; G2 stays open — `ONE-WAY`, never defaulted |
| **Tech lead** | Pick up the quick wins now? Card 1 (after-merge refresh), card 2 (upkeep alert), card 3 (people-link check — verify step only until G2 answers) | **Yes** — cards carry files, steps and acceptance; no further reading needed | Cards 1–2 enter the sprint; card 3's verification query runs (read-only, no gate needed) |

## Your page

| You are | Read |
|---|---|
| CEO | [For the CEO](for-the-ceo.md) |
| COO / Operations | [For operations](for-operations.md) |
| Developers | [For developers](for-developers.md) |

Pass 2 (change briefs, remaining fix cards) and the on-demand pages have not been produced.
Companion audit: [`0.01` company type](../0.01-company-type/README.md).
