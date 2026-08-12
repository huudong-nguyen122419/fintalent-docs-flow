# For operations — `0.01` Define & maintain company type

**The Corporate bucket lies from step 1 — banks and advisories sit in it by definition — and the
moment someone tries to correct or protect a type, the journey dead-ends or fails without telling
anyone.**

## The journey as a person walks it

Pain chips are pinned to the step where they bite, worst first.

| Step | What happens | Status | Pains |
|---|---|---|---|
| **1 · Find companies by type** | Filtering is fast and exact — but the Corporate bucket itself over-includes: banks and advisories sit in the client list | `DEGRADED` | P13, P8 |
| **2 · Check one company** | Open the panel; read type badges, parent and sponsor links | `DEGRADED` | P7 |
| **3 · Fix a wrong type** | While reviewing, the steward spots a mis-tagged firm — there is nowhere to correct it; the bad tag stays | `SILENT-FAIL` | P1 |
| **4 · Refresh from the provider** | Scheduled refreshes rewrite every type value with no warning; the manual refresh button never made it onto the page | `SILENT-FAIL` | P3, P2 |
| **5 · A new company arrives** | A sign-up creates a company with no type — accepted behaviour (2026-08-11); it gains a type on the next provider import | `OK` | — |
| **6 · Bulk tidy-up** | Admins segment with buckets and labels instead; edits apply but leave no record of who changed what | `DEGRADED` | P6, P9, P12 |
| **7 · Auto-classify & report** | The automated classifier tags companies, but low-confidence results hide in logs and four "type" vocabularies disagree in reports | `DEGRADED` | P5, P11 |

P13 is created by step 4's re-derivation but felt at step 1. P10 (one-sided validation of the
lifecycle field) sits beside the flow, on step 2's edit panel.

## Scenarios — confirmed

One happy path plus six variants the code can actually produce, worst first by provisional score.
Confirmed in rev 4; the CPO may still trim before the full journeys are written.

| # | Scenario | Trigger | Persona | Σ RPN |
|---|---|---|---|---|
| **S3** | Provider refresh re-derives type | A provider refresh (today only the scheduled kind — the manual button is disconnected) rewrites all three type values; "Corporate" is recorded as "not sponsor, not portfolio", sweeping banks and advisories into the client bucket | Data steward / system | 134 · P13+P3+P2 |
| **S4** | Correct a misclassified company (dead end) | While reviewing, a steward spots a PE firm tagged Corporate and tries to fix it — there is nowhere to do so; the wrong tag stays | RevOps data steward | 60 · P1 |
| **S5** | Bulk segment selected companies | An admin selects rows and bulk-assigns buckets and labels — the workaround taxonomy; the edit applies instantly but leaves no record of who changed what | RevOps admin | 46 · P6+P9 |
| **S7** | Automated classification backfill | An engineer runs the automated classifier over all companies; results are written safely and repeatably, but low-confidence calls surface only in logs | Platform engineer | 16 · P11 |
| **S2** | Verify one company's classification | Click a row → company panel: type badges, parent / sponsor links, beta classifications; a deleted or merged company shows an empty panel with no explanation | RevOps data steward | 12 · P7 |
| **S1** | Type-filtered prospecting list (main success scenario) | Open the companies list, filter to Sponsors excluding Corporates, page through the results and read the exact count. Counts are exact — but the Corporate bucket itself over-includes (P13, scored under S3) | SDR | 10 · P8 |
| **S6** | Bulk "all matching" with count guard (race / rejection) | An admin applies a bulk edit to "everything matching this filter"; the system rejects it because the matching set changed meanwhile — correct behaviour, but the message offers no way forward | RevOps admin | 8 · P12 |

## Symptom check

Answer the decision by looking, not by memory. Support can forward this table as-is.

| What a user would report | Who hits it | Workaround today |
|---|---|---|
| "My Corporate list is full of banks and M&A advisories." (P13) | Anyone targeting or reporting from the Corporate filter — SDRs, leadership | Cross-check each company's industry (shown on the company panel; industry filters exist) and prune by hand |
| "This company has the wrong type and I can't change it." (P1) | Data stewards reviewing records | None in the product — note it elsewhere; the wrong tag stays |

## Known errors

Not produced. The ITIL known-error table is an on-demand section — request it if support is
affected.
