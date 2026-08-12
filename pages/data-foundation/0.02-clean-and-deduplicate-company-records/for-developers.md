# For developers — `0.02` Remove duplicates & correct company data

Pass 1 ships quick wins only; the rest arrives with pass 2. These cards carry file paths by design.

---

## `QW-1` Clear the duplicate warning the moment a merge lands

**Pain** P1 · **Class** CRITICAL · **Effort** 1 · ≈ 0.5d · `QUICK WIN`

**Why it matters** Every merge — all merges, every admin — currently ends in a false warning for up
to 24h; clearing it instantly costs half a day and removes the top post-merge support question.
**What changes** The merge use-case deliberately leaves `dataHealth` to the nightly scan; clear the
survivor's peer list in the merge write and fire the existing recompute event the way the field-edit
path already does.

**Files**

```
fintalent-backend-microservices:
  apps/microservices/setting/src/modules/mm-company/use-cases/mm-company/merge-company.ts
  apps/gateways/admin/src/modules/mm-company/use-cases/mm-company/merge-mm-company.ts
  pattern to copy: update-v2-fields.ts:163 → mmCompanyEventClient.recomputeDataHealth
```

**Steps**

1. In the micro merge UC (Duplicate branch), after the source stamp: `$set dataHealth.duplicateCandidateIds = []` on the target.
2. Same place: `$pull` the merged `sourceIds` from every other doc's `dataHealth.duplicateCandidateIds` (`updateMany`).
3. In the gateway merge UC, after step 5: emit `recomputeDataHealth({ ids: [companyTargetId] })` — fire-and-forget, like the field-edit path.
4. Unit-test: merge → target verdict has no `SUSPECT_DUPLICATE`.

**Acceptance**

```gherkin
Given two companies flagged as suspected duplicates
When I merge one into the other
Then the survivor shows no duplicate warning without waiting for the nightly scan
And re-opening the review screen shows no phantom candidates
```

**Watch out for** Order matters — clear the peer list *before* the recompute fires: the recompute
derives the `SUSPECT_DUPLICATE` issue from that very field and never writes it. The health
write-guard only protects `dataHealth.*` writes from the assessor path; the dedup-owned peer-list
field is yours to clear here (the detect pass overwrites it nightly anyway).

---

## `QW-2` Alarm when the nightly upkeep doesn't run

**Pain** P4 · **Class** CRITICAL · **Effort** 1 · ≈ 0.5d · `QUICK WIN`

**Why it matters** The entire dedupe and freshness system hangs on two env switches that skip
silently when off — reach is the whole company database, the alert costs half a day.
**What changes** Both schedulers log an info line and return when their flag isn't `"true"`; verify
prod values, then make the skip observable.

**Files**

```
fintalent-backend-microservices:
  apps/gateways/admin/src/schedulers/company/daily-scan-mm-company-data-health.ts   (MM_HEALTH_SCAN_SCHEDULER_ENABLED)
  apps/gateways/admin/src/schedulers/daily-refresh-stale-mm-companies.ts            (MM_REFRESH_STALE_SCHEDULER_ENABLED)
```

**Steps**

1. Check both env flags in production config; record the values in the runbook.
2. In each scheduler's skip branch, replace the info log with a Sentry warning (team convention for scheduler alerts).
3. Add a success log line with counts (both micro UCs already log them) so a missing morning line is itself a signal.
4. Optional: surface "last scan finished at" on the Company health dashboard — the scan already writes a snapshot with that timestamp.

**Acceptance**

```gherkin
Given a nightly upkeep job whose switch is off
When its scheduled time passes
Then the team is alerted the same night
And the health dashboard shows when the last successful scan ran
```

**Watch out for** The flags exist to keep staging and dev replicas from hammering the data provider
— do not remove them; alert only in the environment that is supposed to run. Both jobs are
fire-and-forget events: "emit succeeded" is not "run succeeded" — alert on the micro side's
completion log, not the gateway's.

---

## `QW-3` Make people fully follow the survivor

**Pain** P2 · **Class** BLOCKER · **Effort** 2 · ≈ 1d · `VERIFY-THEN-FIX` — fix behind gate **G2**

**Why it matters** Merged companies may show incomplete people lists — the core promise of merging;
reach is every merge done so far. A read-only count sizes it today, the fix is a day.
**What changes** The merge re-points `workExperiences[].linkedCompany.linkedCompanyId` only; every
contact filter and company-people query reads the top-level `linkedCompanies.linkedCompanyId` array,
which still points at soft-deleted sources.

`UNVERIFIED — whether a pipeline later rebuilds linkedCompanies from work experiences; step 1 settles it.`

**Files**

```
fintalent-backend-microservices:
  apps/microservices/project/src/modules/contact/repositories/contact.repository.ts
    (bulkReassignLinkedCompanyInWorkExperiences, line ~582)
  apps/microservices/project/src/modules/contact/use-cases/contact/bulk-reassign-linked-company.ts
```

**Steps**

1. **Verify (read-only, no gate):** count contacts where `linkedCompanies.linkedCompanyId` points at a company with `parentMergedId` set and `isDeleted`. Zero ⇒ close P2 as handled elsewhere; report where.
2. If > 0 **and G2 approves**: extend the `updateMany` with a second `$set` + arrayFilter for `linkedCompanies.$[lc].linkedCompanyId`.
3. Backfill past merges with the same condition as the verify query — idempotent, re-runnable.
4. De-duplicate: a contact linked to **both** source and target must not end with two identical links.

**Acceptance**

```gherkin
Given a person who worked at a company that was merged away
When I open the surviving company's people and filters
Then that person appears there
And no person appears twice
```

**Watch out for** The backfill rewrites many contact documents — that is why G2 is `ONE-WAY` and
never defaulted. `linkedCompanies` entries carry denormalized flags (`isSponsor`, `hasCampaign`…)
that belong to the company: re-point the id only, never copy flags across. Watch the contact-db sync
pipeline for feedback loops.
