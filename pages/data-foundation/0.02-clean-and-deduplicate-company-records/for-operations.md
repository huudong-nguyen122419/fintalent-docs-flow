# For operations — `0.02` Remove duplicates & correct company data

**The walk from spotting a duplicate to merging it is healthy; the trouble starts the moment the
merge lands, and in the weeks between merges.**

## The journey as an admin walks it

| Step | What happens | Status | Pains |
|---|---|---|---|
| **1 · Spot** | A warning chip on the company row, the health dashboard, or the admin's own eyes flag look-alike records | `OK` | — |
| **2 · Review** | The review screen lays the whole look-alike group side by side, differing fields highlighted (senior admins only — the action is in beta) | `OK` | — |
| **3 · Pick & preview** | Pick the record to keep; a dry-run lists every field where the records disagree and the admin chooses the winning value | `OK` | — |
| **4 · Merge** | Duplicates fold into the survivor: fields combine, people are moved, the change is logged, unsafe re-merges are blocked | `OK` | — |
| **5 · Right after** | The survivor still shows the duplicate warning until tomorrow; some moved people may not appear in its lists; the discarded values are gone without a record | `SILENT-FAIL` | P1, P2, P5 |
| **6 · Weeks later** | Same-name duplicates stay invisible; the nightly upkeep may be off; a merged-away record may be re-imported; ordinary records quietly go stale | `SILENT-FAIL` | P3, P4, P6, P7 |

Each pain, as a person meets it:

| | |
|---|---|
| **P1** | "I merged them — why is it still flagged?" |
| **P2** | While browsing the survivor's people, someone who worked at the duplicate isn't there |
| **P5** | "What did the old record say?" — no answer |
| **P3** | Two same-name companies sit unflagged for months |
| **P4** | The dashboard says "updated daily" while nothing ran |
| **P6** | The duplicate you removed is back after the next import |
| **P7** | A rarely-touched company's data is years old |

## Scenarios — CPO confirms

Confirmed by default after 48h.

1. Spot a flagged duplicate and merge two records (happy path).
2. Merge records whose fields disagree — pick winners in the preview.
3. Merge where people worked at the duplicate — they follow the survivor.
4. Look at the survivor immediately after the merge.
5. Hand-pick duplicates in the list (bulk select, up to 51) and merge.
6. Two same-name companies with nothing else shared — does anything flag them?
7. Correct a wrong field by hand — the health warning updates immediately.
8. An outdated record is refreshed — by button, and by the nightly sweep.

## Symptom check

Answer the decision by looking, not by memory. Support can forward this table as-is.

| What a user would report | Who hits it | Workaround today |
|---|---|---|
| "I merged them but the duplicate warning is still there." (P1) | Any admin, right after a merge | Ignore it — it clears by the next morning. Do not merge again |
| "Someone who worked at the merged company doesn't show under the kept one." (P2) | Ops / sales browsing a company's people | Search for the person by name instead of via the company |
| "These two are obviously the same company — why is nothing flagging them?" (P3) | Data managers | Select both rows in the list and merge by hand (works today) |

## Known errors

Not produced. The ITIL known-error table is an on-demand section — request it if support is
affected.
