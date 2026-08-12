# For the CEO — `0.02` Remove duplicates & correct company data

**Merging duplicates works and is carefully built — but the minutes after a merge confuse people,
one people-link may silently not follow the survivor, and the nightly upkeep can be off without
anyone knowing. Two days of quick wins close the loud gaps.**

## What problem are we solving?

The company database is fed by a data provider, by hand, and by email imports — so the same real
company exists several times, with fields that disagree and records that go stale. Duplicates split
a company's people, campaign history and lists across records; every count and target list built on
them is wrong.

## Where does it stand today?

A real dedupe pipeline exists and the core is good: a nightly scan flags suspected duplicates, a
review screen lets a senior admin pick the survivor, preview every disagreeing field, choose
winners, and merge — people move along, an audit trail records who merged what, and unsafe chains
are blocked. **The merge itself works; the gaps are right after it and in the upkeep around it.**

## What does this deliver?

Pass 1: the decision strip, this summary, the journey health map with scenario list and symptom
check, and three junior-ready fix cards so a dev can start today. Pass 2 after the 48h window: the
change briefs and the remaining fix cards, where the one `ONE-WAY` decision (G2) is made on full
options.

## What will it cost?

Quick wins ≈ **2 dev-days** — after-merge refresh ≈ 0.5d · upkeep alert ≈ 0.5d · people-link verify
and fix ≈ 1d behind gate G2. Remaining ranked debt ≈ **4–7 dev-days**, priced in pass 2. The biggest
item (widening duplicate detection, P3) gets a half-day measurement first, so we only buy it if the
miss-rate justifies it.

## What we would fix

* **`QUICK WIN` · CRITICAL — You merge two records, and the duplicate warning stays.** The survivor
  keeps flashing "suspected duplicate", pointing at records that no longer exist, until an overnight
  job runs. Admins doubt the merge worked, and some try again. **Worth:** trust in the tool restored
  the same minute, for about half a day of work.
* **`QUICK WIN` · CRITICAL — The nightly upkeep can be switched off without anyone noticing.** Both
  overnight jobs — the one that finds duplicates and the one that refreshes stale records — sit
  behind on/off switches that quietly skip when off, while the dashboard keeps saying "updated
  daily". **Worth:** one line of alerting guarantees "updated daily" is actually true, versus weeks
  of silently stale data.
* **BLOCKER · `CONTRACT GAP` — People may not fully follow the survivor.** Each person is linked to
  a company in two places; the merge re-points one of them and leaves the other on the dead record.
  Company people lists, counts and people filters read the untouched one — so someone who worked at
  the merged-away duplicate can vanish from the survivor's view. **Worth:** complete people lists on
  merged companies, the whole point of merging. One verification query tells us how bad it is; the
  fix is ≈ 1 day behind gate G2.
* **CRITICAL · `PRD GAP` — The scanner only recognises two kinds of "same company".** It flags
  records sharing an email domain or a LinkedIn page. Two records with the same name and nothing
  else shared are never flagged, and any group of more than 50 look-alikes is skipped wholesale.
  Nobody knows the miss-rate. **Worth:** a half-day count of invisible same-name pairs tells us
  whether the full fix is worth buying.
* **CRITICAL · `UNVERIFIED` — A merged-away duplicate might come back.** The survivor never inherits
  the duplicate's identity at the data provider — so a survivor created by hand can't be refreshed
  afterwards, and the next provider import may re-create the record we just merged away. The
  re-import path was not fully traced this pass. **Worth:** prevents an endless merge treadmill;
  verify first, fix ≈ 1–2 days if real.
* **CRITICAL — No record of what a merge discarded.** The audit trail says who merged what, not
  which disagreeing values were dropped. "What did that company's phone or address used to say?" is
  unanswerable after the fact. **Worth:** an audit answer for every merge; moderate effort, pass 2.
* **MAJOR — Freshness has a class system.** The weekly refresh sweep covers deal-active companies
  only; ordinary companies and hand-created records go stale indefinitely unless someone remembers
  to click refresh — and hand-created ones have no refresh button at all. **Worth:** define the tier
  we actually need; likely config, not code.

**What already works well:** the review → preview → merge core is careful — every disagreeing field
is shown before committing, the admin picks winners, people move with the merge, unsafe re-merges
are blocked, and each merge is logged. No spend needed here.

## Severity scoreboard

| | Count | |
|---|---|---|
| **Blocker** | 1 | the people-link gap (P2) |
| **Critical** | 5 | 2 are quick wins (P1, P4) |
| **Major** | 1 | freshness tiers (P7) |
| **Quick-win bundle** | ≈ 2d | cards 1–3 |

## Top three quick wins

| # | | Effort |
|---|---|---|
| 1 | **Clear the duplicate warning the moment a merge lands** — refresh the survivor's health flags as part of the merge instead of waiting for tomorrow's scan (P1) | ≈ 0.5d |
| 2 | **Alarm when the nightly upkeep doesn't run** — verify the two switches are on in production and page the team on any skipped night (P4) | ≈ 0.5d |
| 3 | **Make people fully follow the survivor** — one read-only count proves the gap; the re-point fix ships behind gate G2 (P2) | ≈ 1d |

## Impact × effort

Quick-win zone (cheap, high risk): **P1**, **P4**. Expensive and high risk: **P2**, **P3**.
Mid: **P5**, **P6**. Lower: **P7**. Full register with scores and evidence ships on demand.

## What needs a CTO call

| Gate | Type | Question | Recommendation |
|---|---|---|---|
| **G1** | `REVERSIBLE` | Refresh the survivor's health flags as part of every merge, instead of waiting for the nightly scan? | **Yes** — default, executes on silence. Additive; the nightly scan remains the source of truth. *Evidence: the merge deliberately leaves health flags to the next scan.* |
| **G2** | `ONE-WAY` | Also re-point the second people↔company link on merge, and backfill past merges (a rewrite across many people records)? | **Recommend yes — but never defaulted.** Run fix card 3's read-only count first; decide on full options in pass 2. *Evidence: the merge re-points one of the two links; people filters read the other.* |
| **G3** | `REVERSIBLE` | Confirm the two nightly-upkeep switches are on in production and alert on any skipped run? | **Yes** — default, executes on silence. Pure observability. *Evidence: both jobs skip silently with an info-level note when their switch is off.* |
