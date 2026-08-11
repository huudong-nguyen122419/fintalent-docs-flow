# 9 · Build instructions

> part of [0.01 · Define & maintain company type](README.md)

**For: junior/mid devs.** _One card per quick win ([§1](executive-summary.md)). Executable without prior knowledge of the flow. Ordered by RPN, then lowest effort. File paths are exact; acceptance is Gherkin._

{% hint style="info" %}
**Takeaway:** four cards, ≈ 8–12 dev-days total; card 1 waits on G5's mapping (48h default), card 3 on the G4 answer.
{% endhint %}

## 1 · Reclassify non-clients into the new types

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

## 2 · Make company type editable

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

## 3 · Consolidate the type vocabularies

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

## 4 · Audit bulk edits

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
