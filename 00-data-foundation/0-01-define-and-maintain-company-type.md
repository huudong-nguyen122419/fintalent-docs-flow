# 0.01 · Define and maintain company type

## 0.01 · Define & maintain company type

> **0. Data Foundation** · **Lean flow audit · pass 1 + 2 · rev 5**

> **Verdict:** reads work · Corporate over-tags non-clients · 2 silent failures · quick wins ≈ 10 days

Sponsor / PortCo / Corporate classification on company records — plus the five service-firm types added 2026-08-11 (Advisory Firm, Law Firm, Banks, Accounting Firm, Other) — how it is set, shown, filtered, refreshed and (not) corrected. Derived from the connected repos (branch `develop`, read 2026-08-10); dispute any derived value below and it is corrected before pass 2.

Rev 5 adds the product ruling of 2026-08-10 — **Corporate = a Mergermarket-sourced company that could be a Fintalent client**; the code's catch-all derivation conflicts with it, flagged as a PRD gap (P13) with the code as today's reality.

### Who does it

|                | SDR | KAM | Junior Ops | Senior Ops | Automation |
| -------------- | --- | --- | ---------- | ---------- | ---------- |
| **Short term** | R   | ·   | ·          | S          | ·          |
| **Mid-term**   | S   | S   | ·          | R          | S          |

`R` primary owner · `S` supporting · `·` no routine part

### Scope

|                       |                                                                                                                                                                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Flow**              | Classify every company as Sponsor / PortCo / Corporate — service firms as Advisory Firm / Law Firm / Banks / Accounting Firm / Other — and keep that classification correct through data refreshes.                                |
| **Entry points**      | Admin companies list · company detail panel · Data Management (enrichment / health)                                                                                                                                                |
| **Success condition** | The company record carries the correct type; every type-filtered list, count and downstream link reflects it.                                                                                                                      |
| **Frontend**          | `fintalent-microfrontend` · `apps/admin` (companies module + shared data layer)                                                                                                                                                    |
| **Backend**           | `fintalent-backend-microservices` · admin gateway + setting microservice (company module)                                                                                                                                          |
| **Out of scope**      | Dedupe / merge (`0.02`), contact hygiene (`0.05`), campaigns; legacy companies page noted, not traced. **Scenarios: 7.** Requirement = the feature one-liner (`0.01`) + the 2026-08-10 product ruling on the Corporate definition. |

***

## 0 · Decision strip

**For: all stakeholders.** One question each, yes/no. You only veto. <mark style="color:$danger;">**Silence = the recommended default executes after 48h.**</mark>

| Who           | The one question                                                                                                                                | Recommended default                                                                                                                                                                                    | If you do nothing (48h)                                                                          |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| **CEO**       | Approve \~15–18 dev-days to make Corporate mean "possible client", give service firms their own types, and make it all correctable?             | <mark style="color:$success;">**Yes**</mark> — start with the \~10-day quick-win bundle (reclassification · type editor · vocabulary consolidation · bulk audit); schedule the rest behind it.         | The quick-win bundle is scheduled; larger items queue for pass-2 briefs.                         |
| **CPO**       | Sign off the industry → type mapping for the five new types — Advisory Firm, Law Firm, Banks, Accounting Firm, Other (draft mapping at §3, G5)? | <mark style="color:$success;">**Yes**</mark> — adopt the draft mapping; Corporate keeps only possible clients; the scenario list (7 after dropping S8) stays as confirmed.                             | The draft mapping becomes the rule behind fix card 1's backfill — reversible; amend it any time. |
| **COO**       | Do the 2 symptom-check rows at the end of §2 match what support hears (incl. "my Corporate list is full of banks")?                             | <mark style="color:$success;">**Yes**</mark> — rows confirmed by looking; the known-error playbook (§4) stays on demand.                                                                               | Symptom rows stand as written; support can forward them as-is.                                   |
| **CTO**       | Accept the G5 default (industry → type mapping from provider industry data), with G4 held for your §3 answer?                                   | <mark style="color:$success;">**Yes**</mark> — G1 stands as accepted in rev 4 (G2 retired, G3 withdrawn — untyped-at-birth accepted); G5 is **reversible** (re-runnable backfill) and safe to default. | G5 executes with the default mapping; **G4 stays open — ONE-WAY, never defaulted.**              |
| **Tech lead** | Need the technical appendix (surfaces + contract map) before pass 2 starts?                                                                     | <mark style="color:$success;">**Yes**</mark> — a full draft already exists (rev 3 of this audit); it is refreshed and attached before pass 2.                                                          | Appendix ships together with the pass-2 briefs.                                                  |

***

## 1 · Executive summary

**For: CEO.** _Business language, zero code terms, zero raw scores in prose._

> **Takeaway:** finding companies by type works, but **"Corporate" itself is mis-defined** — it sweeps in banks and advisories — and **nobody can fix a wrong type**; four quick wins (≈ 10 dev-days) close the gaps.

### The problem

Every company must be classified Sponsor / PortCo / Corporate — the tag that drives targeting, reporting and account ownership. Today it is set **only** by a data-provider import that computes Corporate as simply _"everything that is neither of the other two"_ — so banks, M\&A advisories and other firms we could never sell to are counted as prospects, and nobody can correct any of it by hand.

### Where it stands today

Finding and filtering companies by type works end-to-end, fast and correct. **Maintaining** the classification does not: there is no way to fix a wrong type and refreshes overwrite silently. Untyped-at-birth for product-created companies is accepted behaviour (2026-08-11).

**Status: reads work; the Corporate bucket over-counts by definition; maintenance fails silently in two places.**

### What this document delivers

**Pass 1:** the decision strip, this summary, and the journey health map with the scenario list (confirmed — explicit go received). **Pass 2** (included below): backend + frontend change briefs (§3) with every change tagged decided-or-CTO, and four junior-ready fix cards (§9). **On demand:** ops playbook, full journeys, technical appendix (drafted in rev 3).

### What it costs

Roughly **15–18 dev-days total**: quick wins ≈ 8–12 dev-days (reclassify service firms into the five new types + backfill, make type correctable, consolidate the type vocabularies, audit bulk edits); the remaining ranked debt ≈ 4–6 dev-days. Refresh-protection and on-demand-refresh work (P2, P3) was removed from scope by product on 2026-08-11; both pains stay on the register.

**Payback:** the quick wins make classification mean what the business means — and make it correctable — for under a week of work.

### The five findings

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

### Quick wins

_Four fixes, ≈ 8–12 dev-days. Build instructions in §9. The eight pains outside this bundle stay on the register below._

|                                                       | Fix                                                                                                                                                                                                                 | Worth                                                                       |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **1** <mark style="color:$danger;background-color:$danger;">**BLOCKER**</mark> | **Redefine "Corporate" = possible client.** Move banks, advisories and law firms into their own new types — Advisory Firm / Law Firm / Banks / Other — via the provider's industry data; backfill existing records. | every type-filtered list and report stops over-counting, for \~2–3 dev-days |
| **2** <mark style="color:$danger;background-color:$danger;">**BLOCKER**</mark> | **Make company type correctable.** No screen or control lets an admin fix a wrong type today.                                                                                                                       | trustworthy target lists and reports, for \~1–3 dev-days                    |
| **3** <mark style="color:orange;background-color:orange;">**CRITICAL**</mark> | **One canonical company type.** Promote the official tag (eight values after win 1); retire the legacy free-text label — gate G4.                                                                                   | cross-team reports finally agree, for \~3–5 dev-days                        |
| **4** <mark style="color:$warning;background-color:$warning;">**MAJOR**</mark>  | **Audit bulk edits.** Bulk edits re-score but skip the record of who changed what.                                                                                                                                  | every bulk action becomes traceable, for \~1–2 dev-days                     |

### Severity scoreboard

| <mark style="color:$danger;background-color:$danger;">BLOCKER</mark> | <mark style="color:orange;background-color:orange;">CRITICAL</mark> | <mark style="color:$warning;background-color:$warning;">MAJOR</mark> | <mark style="color:$info;background-color:$info;">MINOR</mark> |
| :-----------------------------------------: | :-----------------------------------------: | :----------------------------------------: | :-------------------------------------: |
|                    **2**                    |                    **2**                    |                    **5**                   |                  **3**                  |

_12 pain points found, counted per class. Untyped-at-birth (P4) accepted as intended behaviour 2026-08-11 and removed._

### Impact × effort

_Rows = how bad it is when it happens (S5 worst) · columns = how expensive it is to fix (E1 cheapest)._

|        | E1 · <0.5 day      | E2                                                     | E3 · 1–3 days                                               | E4     | E5 · >5 days |
| ------ | ------------------ | ------------------------------------------------------ | ----------------------------------------------------------- | ------ | ------------ |
| **S5** | <mark style="color:$success;background-color:$success;">　</mark> | <mark style="color:$success;background-color:$success;">　</mark>                                                     | <mark style="background-color:$success;">**P1 · P3**</mark> |        |              |
| **S4** | <mark style="color:$success;background-color:$success;">　</mark> | <mark style="background-color:$success;">**P2**</mark> | <mark style="background-color:$success;">**P13**</mark>     |        |              |
| **S3** |                    |                                                        | **P6**                                                      |        | **P5**       |
| **S2** | **P8 · P10 · P12** |                                                        | **P7 · P11**                                                | **P9** |              |
| **S1** |                    |                                                        |                                                             |        |              |

**The green zone** is the top-left corner — hurts a lot, costs little. Anything landing there is quick-win territory by default, and four pains sit in it: **P1, P3, P2, P13**.

**But the quick-win bundle is P13, P1, P5, P6 — not the green four.** Two moved out and two moved in, for reasons the grid cannot see:

| Pain       | Movement                                                                  | Why                                                                                                                     |
| ---------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **P3, P2** | <mark style="background-color:$success;">green zone</mark> → out of scope | descoped by product on 2026-08-11 — a scoring grid does not outrank a product decision; both stay on the register below |
| **P5**     | S3 / E5 → <mark style="color:orange;">quick win 3</mark>                  | the most expensive fix on the board, promoted because every other taxonomy fix waits behind it                          |
| **P6**     | S3 / E3 → <mark style="color:$warning;">quick win 4</mark>                | audit gaps compound — cheap now, unrecoverable later                                                                    |

_Read the grid as the opening argument, not the verdict._

### Pain register

|                                              | Pain                                                                       |    RPN |
| -------------------------------------------- | -------------------------------------------------------------------------- | -----: |
| <mark style="color:$danger;background-color:$danger;">**P1**</mark>   | No way to correct a company's type                                         | **60** |
| <mark style="color:$danger;background-color:$danger;">**P13**</mark>  | Corporate = catch-all; banks & advisories counted as clients · **PRD GAP** | **60** |
| <mark style="color:orange;background-color:orange;">**P3**</mark>    | Refresh silently overwrites classification                                 |     50 |
| <mark style="color:orange;background-color:orange;">**P5**</mark>    | 3+ competing type taxonomies                                               |     36 |
| <mark style="color:$warning;background-color:$warning;">**P6**</mark>  | Bulk edits skip the audit trail                                            |     30 |
| <mark style="color:$warning;background-color:$warning;">**P2**</mark>  | On-demand refresh disconnected                                             |     24 |
| <mark style="color:$warning;background-color:$warning;">**P9**</mark>  | Label list has two sources of truth                                        |     16 |
| <mark style="color:$warning;background-color:$warning;">**P10**</mark> | One-sided lifecycle validation                                             |     16 |
| <mark style="color:$warning;background-color:$warning;">**P11**</mark> | Low-confidence classifications have no review queue                        |     16 |
| <mark style="color:$info;background-color:$info;">**P7**</mark>     | Missing company shows an empty panel                                       |     12 |
| <mark style="color:$info;background-color:$info;">**P8**</mark>     | Bad filter values dropped without error                                    |     10 |
| <mark style="color:$info;background-color:$info;">**P12**</mark>    | Bulk "all matching" rejection offers no recovery guidance                  |      8 |

### Open gates

_Open gates only (G1 accepted in rev 4 · G2 retired · G3 withdrawn — untyped-at-birth accepted). REVERSIBLE defaults execute on silence; ONE-WAY gates wait for §3._

| Gate                | Question                                                                                                                                                                                  | Auditor recommendation                                                                                                                                      |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **G5** <mark style="color:$success;background-color:$success;">REVERSIBLE</mark> | Adopt the five new service-firm types — Advisory Firm, Law Firm, Banks, Accounting Firm, Other — mapped from the provider's own industry data, leaving Corporate = possible clients only? | **Yes** — start with the draft mapping (gate G5, §3); the backfill is re-runnable, so an amended mapping re-applies cleanly. **Executes on silence (48h).** |
| **G4** <mark style="color:$danger;background-color:$danger;">ONE-WAY</mark>    | Taxonomy end-state: which vocabulary is canonical for "company type"?                                                                                                                     | Promote the official type tag and retire the legacy text label — now gates quick-win card 3. **No silence default** — answer on the §3 options.             |

***

## 2 · Journey health map

**For: COO + CPO.** _The flow as a person walks it. Plain words — read this aloud in standup._

> **Takeaway:** the Corporate bucket lies from step 1 — banks and advisories sit in it by definition — and the moment someone tries to correct or protect a type, the journey dead-ends or fails without telling anyone.

### The seven steps

| Step                            | What happens                                                                                                                       | Status                                                                       | Pains                                                                                                              |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **1** Find companies by type    | Filtering is fast and exact — but the Corporate bucket itself over-includes: banks and advisories sit in the client list.          | <mark style="color:$warning;background-color:$warning;">DEGRADED</mark>      | <mark style="color:$danger;background-color:$danger;">P13</mark> · <mark style="background-color:$info;">P8</mark> |
| **2** Check one company         | Open the panel; read type badges, parent and sponsor links.                                                                        | <mark style="color:$warning;background-color:$warning;">DEGRADED</mark>      | <mark style="background-color:$info;">P7</mark>                                                                    |
| **3** Fix a wrong type          | While reviewing, the steward spots a mis-tagged firm — there is nowhere to correct it; the bad tag stays.                          | <mark style="color:$danger;background-color:$danger;">**SILENT-FAIL**</mark> | <mark style="color:$danger;background-color:$danger;">P1</mark>                                                    |
| **4** Refresh from the provider | Scheduled refreshes rewrite every type value with no warning; the manual refresh button never made it onto the page.               | <mark style="color:$danger;background-color:$danger;">**SILENT-FAIL**</mark> | P3 · P2                                                                                                            |
| **5** A new company arrives     | A sign-up creates a company with no type — accepted behaviour (2026-08-11); it gains a type on the next provider import.           | <mark style="color:$success;background-color:$success;">OK</mark>            | —                                                                                                                  |
| **6** Bulk tidy-up              | Admins segment with buckets/labels instead; edits apply but leave no record of who changed what.                                   | <mark style="color:$warning;background-color:$warning;">DEGRADED</mark>      | P6 · P9 · P12                                                                                                      |
| **7** Auto-classify & report    | The automated classifier tags companies, but low-confidence results hide in logs and four "type" vocabularies disagree in reports. | <mark style="color:$warning;background-color:$warning;">DEGRADED</mark>      | P5 · P11                                                                                                           |

_Steps 1→7 left to right as the journey runs. P13 is created by step 4's re-derivation but felt at step 1; P10 (one-sided validation of the lifecycle field) sits beside the flow on step 2's edit panel._

### Scenarios

<mark style="color:$success;background-color:$success;">CONFIRMED</mark> _in rev 4; CPO may still trim before §5 expansion. One happy path + six variants the code can actually produce, worst first by provisional score._

| #      | Scenario                                                  | Trigger                                                                                                                                                                                                                                | Persona               |               Σ RPN |
| ------ | --------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ------------------: |
| **S3** | Provider refresh re-derives type                          | A provider refresh (today only the scheduled kind — the manual button is disconnected) rewrites all three type values; "Corporate" is recorded as "not sponsor, not portfolio" — sweeping banks and advisories into the client bucket. | Data steward / system | **134** · P13+P3+P2 |
| **S4** | Correct a misclassified company _(dead end)_              | While reviewing, a steward spots a PE firm tagged Corporate and tries to fix it — there is nowhere to do so; the wrong tag stays.                                                                                                      | RevOps data steward   |         **60** · P1 |
| **S5** | Bulk segment selected companies                           | An admin selects rows and bulk-assigns buckets/labels — the workaround taxonomy; the edit applies instantly but leaves no record of who changed what.                                                                                  | RevOps admin          |      **46** · P6+P9 |
| **S7** | Automated classification backfill                         | An engineer runs the automated classifier over all companies; results are written safely and repeatably, but low-confidence calls surface only in logs.                                                                                | Platform engineer     |        **16** · P11 |
| **S2** | Verify one company's classification                       | Click a row → company panel: type badges, parent / sponsor links, beta classifications; a deleted or merged company shows an empty panel with no explanation.                                                                          | RevOps data steward   |         **12** · P7 |
| **S1** | Type-filtered prospecting list _(main success scenario)_  | Open the companies list, filter to Sponsors excluding Corporates, page through the results and read the exact count. Counts are exact — but the Corporate bucket itself over-includes (P13, scored under S3).                          | SDR                   |         **10** · P8 |
| **S6** | Bulk "all matching" with count guard _(race / rejection)_ | An admin applies a bulk edit to "everything matching this filter"; the system rejects it because the matching set changed meanwhile — correct behaviour, but the message offers no way forward.                                        | RevOps admin          |         **8** · P12 |

### Symptom check

_Answers the COO's §0 row by looking, not memory. Support can forward this as-is._

| What a user would report                                          | Who hits it                                                                | Workaround today                                                                                            |
| ----------------------------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| _"My Corporate list is full of banks and M\&A advisories."_ <mark style="color:$danger;background-color:$danger;">P13</mark> | Anyone targeting or reporting from the Corporate filter (SDRs, leadership) | Cross-check each company's industry (shown on the company panel; industry filters exist) and prune by hand. |
| _"This company has the wrong type and I can't change it."_ <mark style="color:$danger;background-color:$danger;">P1</mark>   | Data stewards reviewing records                                            | None in the product — note it elsewhere; the wrong tag stays.                                               |

***

## 3 · Change briefs

**For: CTO.** _Change contracts, not implementations. Every change is tagged <mark style="color:$success;background-color:$success;">DECIDED</mark> or <mark style="color:$warning;background-color:$warning;">NEEDS CTO</mark> and traced to a pain ID. JSON samples live only here. Exact file evidence: technical appendix (rev 3)._

> **Takeaway:** four changes make company type mean the right thing, writable, audited and single-vocabulary; one call remains yours — the taxonomy end-state (G4), now gating quick-win card 3; the industry → type mapping (G5) defaults in 48h.

### 3A · Backend change brief

**Current status:** one company store (setting service) holds three independent yes/no type values plus parallel vocabularies (legacy text label, automated tags, buckets, labels). Writes arrive through a single-record edit (whitelisted fields, audited), a bulk edit (count-guarded, unaudited), and a provider-refresh job (overwrites everything, derives "Corporate" as the complement of the other two flags, manual trigger disconnected). Every write already re-scores company data health automatically.

#### 3A-B · Database updates

| Store           | Change needed                                                                                                                                                                                                                                                                                                         | Why      | Migration risk                                                                                                                                                                 | Decision                                                              |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| Company records | No schema change for editing — the three type values already exist; only the write whitelist widens.                                                                                                                                                                                                                  | P1 · G1  | None.                                                                                                                                                                          | <mark style="color:$success;background-color:$success;">DECIDED</mark>                                                             |
| Audit log store | No new table — bulk writes reuse the existing per-field change-history records the single edit already produces.                                                                                                                                                                                                      | P6       | Low — write volume rises on bulk actions; events are batched.                                                                                                                  | <mark style="color:$success;background-color:$success;">DECIDED</mark>                                                             |
| Company records | Add five new type values — **Advisory Firm, Law Firm, Banks, Accounting Firm, Other** — following the existing independent-flag pattern; the provider's industry evidence (sector, industry hierarchy, NAICS codes) already stored per company drives them; a **re-runnable backfill** reclassifies existing records. | P13 · G5 | Low–medium — additive flags, deterministic from stored fields; the search index gains the new fields (one rebuild window needed); a re-run applies an amended mapping cleanly. | <mark style="color:$success;background-color:$success;">DECIDED</mark> mechanics; the industry → type mapping is G5 (48h default). |

#### 3A-C · API updates

**Edit one company** · existing operation, widened · <mark style="color:$success;background-color:$success;">DECIDED · G1</mark>

Accepts all type values — the existing three plus Advisory Firm / Law Firm / Banks / Accounting Firm / Other (`true` / `false` / `null-to-clear`) — and writes the audit record — resolves P1. With refresh protection descoped (P3), the next provider refresh may overwrite a manual value. Back-compat: additive input fields, non-breaking, no version bump.

```graphql
# request
mutation updateMmCompanyV2Fields
{ "id": "663d…9f1",
  "input": {
    "isSponsor": true,
    "isCorporate": false } }
```

```json
// success
{ "data": { "updateMmCompanyV2Fields": {
    "id": "663d…9f1", "isSponsor": true,
    "isPortfolio": false, "isCorporate": false } } }

// error (validation)
{ "errors": [ { "message": "isSponsor must be a boolean",
    "extensions": { "code": "BAD_USER_INPUT" } } ] }
```

**Bulk edit companies** · existing operation, widened + audited · <mark style="color:$success;background-color:$success;">DECIDED</mark>

Gains the same three type fields and emits the per-company audit records the single edit already produces — resolves P6, enables bulk type assignment. The count-guard rejection stays; its message gains recovery guidance (P12, copy in 3B-c). Back-compat: additive, non-breaking.

```graphql
# request
mutation bulkUpdateMmCompanyV2Fields
{ "condition": { "ids": ["663d…9f1", "663d…a02"] },
  "input": { "isCorporate": true } }
```

```json
// success
{ "data": { "bulkUpdateMmCompanyV2Fields": true } }

// error (count drift on "all matching")
{ "errors": [ { "message": "count mismatch",
    "extensions": { "code": "MM_COMPANY_DIFFERENT_COUNT_QUANTITY" } } ] }
```

#### 3A-D · Decision gates

| Gate   | Rule / question                                                                                               | Options considered                                                                                                                                                                                                                                                                                                                          | Status                                                                                                                                                                                                                                                                                                  |
| ------ | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **G1** | Who may edit company type; are the three values exclusive?                                                    | a) all admins, single-select · b) senior data roles, independent values                                                                                                                                                                                                                                                                     | <mark style="color:$success;background-color:$success;">DECIDED</mark> — **(b)**: matches stored data and the existing role model; accepted via the §0 strip.                                                                                                                                                                                                        |
| **G2** | Do manual type fixes survive provider refreshes?                                                              | a) provider always wins · b) admin-owned values protected via provenance                                                                                                                                                                                                                                                                    | <mark style="color:$info;background-color:$info;">DESCOPED</mark> — product removed the refresh-protection work on 2026-08-11; today's behaviour stays (a) provider always wins. P3 remains on the pain register.                                                                                                                                              |
| **G3** | Give untyped companies a visible home + backfill?                                                             | a) leave invisible · b) untyped filter now, classifier backfill after                                                                                                                                                                                                                                                                       | <mark style="color:$info;background-color:$info;">WITHDRAWN</mark> — untyped-at-birth accepted as intended behaviour (2026-08-11); no untyped filter or classifier backfill ships.                                                                                                                                                                             |
| **G5** | Corporate = client-eligible only (PRD gap P13): which provider industries map to which of the five new types? | a) draft mapping on the provider's sector/industry codes — commercial & investment banks → Banks; M\&A / corporate-finance advisories, consultancies → Advisory Firm; accountancy & audit → Accounting Firm; law firms → Law Firm; other non-client service industries → Other · b) coarser split — Banks plus one combined "Services" type | <mark style="color:$warning;background-color:$warning;">DEFAULTS 48H</mark> — auditor recommends (a), reviewed quarterly; REVERSIBLE via the re-runnable backfill. Reclassified firms leave the Corporate filter by design; unmapped industries stay Corporate. Companies with no industry data stay Corporate and are logged for review. CPO owns the mapping (§0). |
| **G4** | Taxonomy end-state: which vocabulary is canonical for "company type"?                                         | a) promote the official type tag (now eight values), retire the legacy text label · b) adopt the automated two-level classification as canonical                                                                                                                                                                                            | **<mark style="color:$warning;background-color:$warning;">NEEDS CTO</mark>** — auditor recommends (a) now, revisit (b) once a review queue exists (P11). Resolves P5; gates quick-win card 3.                                                                                                                                                                        |

#### 3A-E · Completion checklist _(verification only)_

* Setting a type value via single edit is visible on re-read, in the list row, and in the change history (P1).
* A bulk edit of N companies produces N history entries identical in shape to single-edit entries (P6).
* After the backfill, a provider company in a mapped service industry (e.g. an M\&A advisory) is tagged Advisory Firm and absent from the Corporate filter, while a client-industry peer keeps its Corporate tag (P13).
* Backend coverage modes each covered-by-test or explicitly waived: validation rejection, authz rejection, dependency failure, timeout/retry, double submit, partial failure.
* Contract map (rev 3, 7c) re-run: rows 1 and 2 clear; no new gaps; no new swallowed errors introduced.

### 3B · Frontend change brief

**Current status:** the companies list and company panel show type as read-only badges; admins segment with buckets/labels instead; a finished refresh button exists but was never placed on a screen; there is no view of untyped companies and no explanation when a panel opens on a merged/removed record.

#### 3B-B · API surface for the frontend

| Operation                | Status       | What the frontend must change                                                                                                                               | Replaces                                                                                                                         | Pain     |
| ------------------------ | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------- |
| Edit one company         | <mark style="color:$info;background-color:$info;">CHANGED</mark>    | Add a role-gated "Company type" editor row in the company panel (same inline pattern as lifecycle); optimistic badge update; extend the editable-field set. | Read-only badges as the only type surface.                                                                                       | P1       |
| Bulk edit companies      | <mark style="color:$info;background-color:$info;">CHANGED</mark>    | Add "Set company type" to the bulk action bar (reuses the bucket/labels panel pattern); improve the count-drift rejection message (copy below).             | Bucket/labels as the only bulk taxonomy.                                                                                         | P6 · P12 |
| List filter              | <mark style="color:$info;background-color:$info;">CHANGED</mark>    | Add "Advisory Firm", "Law Firm", "Banks", "Accounting Firm" and "Other" values to the company-type filter chips; counts come from the existing facet call.  | —                                                                                                                                | P13      |
| Legacy text type display | <mark style="color:$info;background-color:$info;">DEPRECATED</mark> | Stop presenting the legacy free-text type on legacy surfaces once G4 is answered.                                                                           | Replaced by the official type tag everywhere. Removal note: hide after G4 decision; field stays in storage until pass-3 cleanup. | P5       |

#### 3B-C · Changed journey + UX copy

**Fix-a-type journey**

1. **Open the company panel** — type row now shows badges + an Edit control (permitted roles).
2. **Change the type** — pick from all eight types (incl. Advisory Firm / Law Firm / Banks / Accounting Firm / Other); save inline; badges update instantly.
3. **Recorded** — change lands in the company's history with the admin's name.

**Proposed copy**

| Surface                        | Current copy                                                       | Proposed copy                                                                                 | State          | Decision  |
| ------------------------------ | ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- | -------------- | --------- |
| Company panel — type row       | _(badges only, no label or control)_                               | "Company type" · helper: _"May be overwritten by the next provider refresh."_                 | Default / edit | <mark style="color:$success;background-color:$success;">DECIDED</mark> |
| Bulk "all matching" rejection  | Raw error toast surfacing the server message `UNVERIFIED verbatim` | _"The matching list changed while you were editing. Review the updated count and try again."_ | Error toast    | <mark style="color:$success;background-color:$success;">DECIDED</mark> |
| Type filter chips              | "Sponsor · Portfolio · Corporate"                                  | Add chips: _"Advisory Firm · Law Firm · Banks · Accounting Firm · Other"_                     | Filter bar     | <mark style="color:$success;background-color:$success;">DECIDED</mark> |
| Company panel — missing record | _(empty panel, no message)_                                        | _"This company was merged or removed. Open the surviving record from its parent link."_       | Empty state    | <mark style="color:$success;background-color:$success;">DECIDED</mark> |

#### 3B-D · Completion checklist _(verification only)_

* Type editor visible only to permitted roles; read-only roles see badges unchanged.
* Each changed screen covers happy / empty / loading / error states.
* Full journey walk: list → filter → panel → edit type → badge + history reflect it.
* Deep-link, back and refresh keep filter state; concurrent-edit pass shows the improved rejection message.

***

## 9 · Build instructions

**For: junior/mid devs.** _One card per quick win (§1). Executable without prior knowledge of the flow. Ordered by RPN, then lowest effort. File paths are exact; acceptance is Gherkin._

> **Takeaway:** four cards, ≈ 8–12 dev-days total; card 1 waits on G5's mapping (48h default), card 3 on the G4 answer.

### 1 · Reclassify non-clients into the new types

<mark style="color:$danger;background-color:$danger;">P13 · RPN 60 · BLOCKER · PRD GAP</mark> · `E3 ≈ 2–3 days`

**PM:** Banks, advisories and law firms leave the Corporate prospect bucket and become their own filterable types (Reach 3 × Impact 5, Confidence High on the mechanism) — the definition the business ruled on. Waits only on the G5 mapping (48h default).

**Engineering:** add the five service-type flags; replace the complement derivation ("not sponsor, not portfolio") with the industry → type mapping keyed on fields already stored per record; backfill existing records; extend the index and filter chips.

**Files**

* `apps/microservices/setting/src/modules/mm-company/use-cases/mm-company/refresh-from-mergermarket-v2.ts` (the `isCorporate` complement line in `upsertCompany`)
* a new industry → type mapping constant beside it (new file, e.g. `use-cases/mm-company/company-type-mapping.ts`)
* backfill following the existing field-recompute pattern in `migrations-v2/company/fields/`
* industry evidence per record: `mmSector` · `industryBig/Gig/Sig` · `primaryNaicsCode` · `naicsCode2/3Digits` (`libs/schema/src/setting/mm-company.schema.ts`)
* new flags land in that schema + `libs/enums/src/setting/mm-company.enum.ts`
* filter chips + condition mapping in `apps/admin/src/modules/mm-company-v2`

**Steps**

1. Add the five flags (Advisory Firm, Law Firm, Banks, Accounting Firm, Other) to the company schema and enums.
2. Land the signed G5 mapping as a reviewed constant keyed on provider sector/industry codes (NAICS 2/3-digit as fallback).
3. In the refresh upsert, derive Sponsor/Portfolio as today; mapped industries set their service flag (incl. Other for non-client industries outside the three named types); the unmapped remainder stays Corporate.
4. Unit-test nine cases: sponsor · portfolio · client corporate · bank · advisory · law firm · accounting firm · other · no-industry-data (stays Corporate, logged).
5. Write the re-runnable backfill reclassifying active records (batch write + one batched health-recompute emit, mirroring the existing recompute migrations).
6. Extend the search index and type filter chips; run on staging and review before/after counts per type with the CPO.

**Acceptance:** _Given_ a provider company that is neither Sponsor nor Portfolio and whose industry maps to a service type (e.g. an M\&A advisory) · _When_ the backfill or a fresh provider refresh runs · _Then_ it is tagged Advisory Firm, leaves the Corporate filter and appears under the new chip, while a client-industry peer keeps its Corporate tag.

**Watch out for:** the derivation reruns on every refresh — with refresh protection descoped (P3), a manual reclassification is re-derived away on the next scheduled refresh, so the mapping is the only durable control; and the new index fields need one rebuild window — schedule it with the CTO.

### 2 · Make company type editable

<mark style="color:$danger;background-color:$danger;">P1 · RPN 60 · BLOCKER</mark> · `E2 ≈ 1–3 days`

**PM:** Reaches every data steward and everyone downstream of their lists (Reach 3 × Impact 5, Confidence High) for effort 2 — the highest-return change in the audit.

**Engineering:** widen the single-edit input whitelist to the three type flags (BE) and add a role-gated inline type editor to the company panel (FE).

**Files**

* `apps/gateways/admin/src/modules/mm-company/domain/inputs/update-mm-company-v2-fields.resolver.input.ts`
* `apps/gateways/admin/src/modules/mm-company/use-cases/mm-company/mm-company-v2-field-config.ts`
* `libs/fintalent-shared/src/use-cases/admin/mm-company-v2/use-update-mm-company-v2-fields.ts`
* `apps/admin/src/modules/mm-company-v2/presenters/components/organisms/company-detail-drawer/left-panel/company-details-panel.tsx`

**Steps**

1. Add `isSponsor` / `isPortfolio` / `isCorporate` (nullable booleans) to the input class.
2. Add the same keys to the tracked + scalar field lists in the field config.
3. Extend the FE editable-field union and mutation types.
4. Add a role-gated "Company type" editor row in the panel (reuse the lifecycle inline pattern).
5. Patch the row store from the mutation response.

**Acceptance:** _Given_ an admin with company-edit rights viewing a company tagged Corporate · _When_ they set Sponsor on and save · _Then_ the badges update instantly, the company's history lists the change with their name, and a reload still shows Sponsor.

**Watch out for:** the input class and the tracked-field config must change together — a field missing from the tracked list is silently dropped from the write. Provider refreshes still overwrite manual type values (P3 descoped) — the helper copy in 3B-c warns stewards.

### 3 · Consolidate the type vocabularies

<mark style="color:orange;background-color:orange;">P5 · RPN 36 · CRITICAL</mark> · `E4 ≈ 3–5 days`

**PM:** One canonical "company type" ends hand-reconciled cross-team reports (Reach 3 × Impact 3, Confidence Med). Waits on G4 — the one ONE-WAY gate; the auditor recommends promoting the official tag and retiring the legacy text label.

**Engineering:** make the official type tag (eight values after card 1) the only vocabulary on admin surfaces and reports; hide the legacy free-text label; keep automated classifications as beta-only advisory signals.

**Files**

* `company-status-tags` + `company-labels-row` components in `apps/admin/src/modules/mm-company-v2`
* legacy text field in `libs/schema/src/setting/mm-company.schema.ts` + `libs/enums/src/setting/mm-company.enum.ts`
* automated classifier output in `apps/microservices/setting/src/modules/mm-company/use-cases/mm-company/update-categorize.ts`

**Steps**

1. Get G4 answered (recommended: official tag canonical).
2. Hide the legacy free-text type on admin surfaces behind a flag — the stored field stays for pass-3 cleanup.
3. Downgrade automated classifications to advisory-only beta chips beside the official tag.
4. Sweep list columns, filters and reports to read only the official flags.
5. Announce the change to teams using buckets/labels as a workaround taxonomy.

**Acceptance:** _Given_ a company whose legacy text type disagrees with its official tag · _When_ an admin views the list, panel or a report · _Then_ only the official tag drives what is shown and counted, and the legacy text no longer appears.

**Watch out for:** ONE-WAY — do not start before the CTO signs G4; hide, don't delete, the legacy field.

### 4 · Audit bulk edits

<mark style="color:$warning;background-color:$warning;">P6 · RPN 30 · MAJOR</mark> · `E2 ≈ 1–2 days`

**PM:** Closes the _"who changed this?"_ gap for every bulk action (Reach 2 × Impact 3, Confidence High) for effort 2.

**Engineering:** emit the same per-field change-history event from the bulk path that the single edit already emits — batched, with actor info.

**Files**

* `apps/gateways/admin/src/modules/mm-company/use-cases/mm-company/bulk-update-v2-fields.ts` (both the ids and by-condition branches)
* consumer already exists in the log microservice (`company-history` module)

**Steps**

1. After the batch write, collect the affected company ids.
2. Emit one batched fields-updated history event carrying actor id/name and the changed fields.
3. Mirror for the async by-condition branch using its resolved ids.
4. Verify the log service ingests both.
5. Spot-check the history panel on a bulk-edited company.

**Acceptance:** _Given_ an admin bulk-assigns a bucket to 20 companies · _When_ the write completes · _Then_ each of the 20 histories shows the change with the admin's name and timestamp.

**Watch out for:** emit ONE batched event, not a per-company loop — the existing health-recompute emit shows the batch pattern to copy.

***

_Removed from scope by product (2026-08-11): **Protect manual types from refresh** (P3 · RPN 50) and **Reconnect the refresh button** (P2 · RPN 24) — both pains stay on the register; their full cards are preserved in rev 4. Rev 4's **Enforce lifecycle validation** card (P10 · RPN 16 · E1) also sits in the pass-2 backlog, unchanged._
