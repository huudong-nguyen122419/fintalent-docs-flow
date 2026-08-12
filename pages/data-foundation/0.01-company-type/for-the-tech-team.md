# For the tech team — `0.01` Define & maintain company type

**The only page in this activity carrying file paths, handlers and field names.** It documents the
`companyClassification` migration: one scalar enum replaces `isSponsor` / `isPortfolio` /
`isCorporate` plus the five service-firm flags — schema, industry mapping, priority-collapse
backfill, the two mutations, the frontend surfaces, and the four-phase rollout.

**4 phases · expand-contract · G6 priority order still open · backend ≈ 4–6 dev-days · frontend ≈ 3–4 dev-days**

---

## 1 · Schema & enum

One new field, one new enum, both additive. Nothing existing is removed in this phase.

`libs/enums/src/setting/mm-company.enum.ts`

```ts
export enum CompanyType {
  Sponsor = 'Sponsor',
  Portfolio = 'Portfolio',
  Bank = 'Bank',
  AdvisoryFirm = 'AdvisoryFirm',
  LawFirm = 'LawFirm',
  AccountingFirm = 'AccountingFirm',
  Corporate = 'Corporate',
  Other = 'Other',
}
```

PascalCase (2026-08-11); PA and Agency withdrawn as distinct values — both classify as `Other`.
**The declaration order IS the priority order (G6)** — the backfill imports this array directly, so
keep it meaningful.

`libs/schema/src/setting/mm-company.schema.ts`

```ts
@Prop({ type: String, enum: CompanyType, index: true, default: null })
companyClassification?: CompanyType | null;

// deprecated — kept through Phase 3
@Prop({ type: Boolean }) isSponsor?: boolean;
@Prop({ type: Boolean }) isPortfolio?: boolean;
@Prop({ type: Boolean }) isCorporate?: boolean;
// + isAdvisoryFirm / isLawFirm / isBank / isAccountingFirm / isOther
```

**GraphQL** — register `CompanyType` as an enum type; add `companyClassification` to `MmCompanyV2`,
`UpdateMmCompanyV2FieldsInput` and `BulkUpdateMmCompanyV2FieldsInput`. No field is removed from any
input or output type in this pass: additive schema change, no client breaks.

## 2 · Industry-based classification (G5, made concrete)

How the five service-firm flags and `Bank` get their starting value from the industry hierarchy
already stored per company, before the collapse ever runs. Source taxonomy:
`mm-company-sectors.json` (frontend repo) — 10 sectors, ~90 groups, ~200 leaf subIndustries.

**Rule 0 — never override a manual value.** A company whose `companyClassification` was last set by
a human (per change-history) is skipped entirely by industry classification. Only untyped or
provider-derived records are eligible — the same spirit as the descoped P3 refresh protection,
applied narrowly to this one pass.

| Sector | Group | SubIndustry | → Type | Confidence |
|---|---|---|---|---|
| Business Services | Professional Services | Legal | `LawFirm` | MATCHED |
| Business Services | Professional Services | Accounting | `AccountingFirm` | MATCHED |
| Business Services | Professional Services | Management Consulting | `AdvisoryFirm` | MATCHED |
| Financial Institutions | Finance | Investment Banks | `Bank` | MATCHED |
| Financial Institutions | Finance | Commercial & Savings Banks | `Bank` | MATCHED |
| Financial Institutions | Finance | Municipal / Provincial Banks | `Bank` | MATCHED |
| Business Services | Professional Services | Personnel | `Other` | MATCHED — Agency withdrawn; staffing/search classifies as Other |
| Financial Institutions | Finance | Investment Management | `Corporate` | NO INDUSTRY SIGNAL — Sponsor/Portfolio come from Mergermarket deal data, not the sector taxonomy; falls through to Corporate unless MM data says otherwise |
| Financial Institutions | Finance | Acquisitions/Restructurings | `AdvisoryFirm` | MATCHED |
| Financial Institutions | Insurance | Brokers | `Other` | MATCHED |
| — all other subIndustries (≈ 190) — | Manufacturing, retail, healthcare, tech, real estate, transport, government, remaining finance/insurance | `Corporate` | NO SIGNAL — falls through to the existing complement rule |

**Gap to flag with the CPO:** PA and Agency were withdrawn as distinct types (2026-08-11), so
placement-agent and staffing/search signals classify as `Other`. **Sponsor and Portfolio are never
derived from the sector taxonomy** — both come from Mergermarket deal-role data (the existing
`isSponsor` / `isPortfolio` flags, unchanged by this migration). The industry map only ever assigns
the other six values.

The mapping lives as a reviewed constant (`SUBINDUSTRY_TO_TYPE`) keyed on `subIndustry.name`, beside
the refresh use-case. Classification sets the underlying boolean flag (e.g. `isLawFirm = true`); the
collapse then reduces whichever flags are set into the single value. A company with both `isSponsor`
and an industry-derived `isLawFirm` still resolves to `Sponsor` — industry signal never outranks an
existing capital-side flag.

## 3 · Priority-collapse backfill

`apps/microservices/setting/src/modules/mm-company/use-cases/mm-company/backfill-company-type.ts` **(new)**

Runs once per environment before Phase 2 opens; re-runnable and idempotent so an amended priority
order (pending G6) re-applies cleanly. `collapse()` walks `Object.values(CompanyType)` in
declaration order and returns the first type whose flag is true; no flag true → `null` (untyped, per
the P4 ruling).

**Steps**

1. Batch-read companies (cursor, 500/page) with any legacy flag true, or with an already-stale `companyClassification` (for re-runs).
2. Run `collapse()`; write `companyClassification` only where the value changed.
3. Write a single change-history entry per company: field `companyClassification`, note listing **all** true flags that were collapsed — not just the winner. This is the "nothing unrecoverable" record.
4. Log a per-run summary: total scanned, changed, unchanged, newly-untyped, by-type histogram.
5. `--dry-run` prints the histogram without writing — run this first in production and attach the output to the G6 approval.

**Watch out for**

* Re-running after G6 changes the order must not touch companies with a manual `companyClassification` edit recorded post-backfill — check change-history for a manual entry before overwriting.
* The five service-firm flags land from the G5 industry mapping on their own schedule; this script must tolerate running before or after that backfill. A company with none of the eight flags is legitimately untyped, not an error.
* Batch size 500 with a small delay between batches — this is the same collection the nightly dedupe and health scans read ([`0.02`](../0.02-clean-and-deduplicate-company-records/README.md)); avoid overlapping a scan window.

## 4 · Mutations

Both existing operations gain `companyClassification`; neither runs priority logic — that happens
once, in the backfill. A manual edit is always a plain replace.

| File | Change | Notes |
|---|---|---|
| `apps/gateways/admin/.../update-mm-company-v2-fields.ts` | Whitelist `companyClassification`; validate against the enum; on write, also set the matching legacy flag true and all others false (dual-write, Phase 2) so old readers keep working | Reuses the existing per-field audit write — no new audit code. Resolves P1 |
| `apps/microservices/setting/.../bulk-update-v2-fields.ts` | Same whitelist + dual-write; count-guard and per-company history emission unchanged (already batches). `MM_COMPANY_DIFFERENT_COUNT_QUANTITY` gains a recovery-guidance message (P12) | Resolves P6; one value applied to every matched id, no merge semantics on a scalar |
| `apps/microservices/setting/.../refresh-from-mergermarket-v2.ts` | Provider refresh keeps writing the legacy flags only in Phases 1–2; once Phase 3 flips reads, this UC must also call `collapse()` so a refresh doesn't leave `companyClassification` stale | **Easy to miss** — the one write path outside the two mutations; flag it explicitly in the Phase 3 PR. Provider refresh overwriting a manual value is intentional (P3 descoped) |

### Error-path coverage

| Mode | How this migration covers it |
|---|---|
| Validation rejection | Unknown enum value → GraphQL `BAD_USER_INPUT`. Covered by test |
| Authz rejection | Reuses the existing field-whitelist authz on both mutations — no new authz path. Covered by existing test |
| Dependency failure | A change-history write failure must not roll back the `companyClassification` write (matches existing single-edit behaviour) — same transaction as today, no new dependency. Covered by existing test |
| Timeout / retry | The backfill is idempotent and re-runnable by design; mutations add no new retry surface. **Waived** |
| Double submit | A scalar replace is naturally idempotent — double-submitting writes the same state twice. **Waived** |
| Partial failure (bulk) | Existing bulk batching and partial-failure behaviour is unchanged by the field swap. Covered by existing test |

## 5 · Rollout — expand / dual-write / contract

Each phase is its own release. No phase depends on the frontend being ready: both writers and both
readers exist simultaneously through Phase 2.

| # | Phase | What ships | Exit condition |
|---|---|---|---|
| 1 | **Expand** | Schema + enum ship; field is `null` everywhere; no reader or writer touches it | Deployed with zero behaviour change; safe to ship any time |
| 2 | **Backfill + dual-write** | Priority-collapse backfill runs; both mutations write both shapes; provider refresh still legacy-only. Search index gains the `companyClassification` field — **schedule the rebuild window with the CTO alongside this phase, not Phase 4**; the field must be queryable before Phase 3 flips list filters to it | Backfill histogram reviewed against G6; 100% of active companies have a non-stale value or are legitimately untyped; index rebuild confirmed queryable in staging |
| 3 | **Flip reads** | Frontend reads and writes `companyClassification` exclusively; list filters, facets and search index switch to the enum field; refresh UC gains `collapse()` | No production traffic reads the legacy flags for a full monitoring window (≥ 1 week, including one weekly refresh cycle) |
| 4 | **Contract** | Drop the eight legacy flags from schema, GraphQL types and dual-write code; remove them from the search index mapping | Own release, reviewed separately — never bundled with feature work |

**G6 must close before Phase 2's backfill runs in production.** The dry-run output is produced either
way and doesn't need the gate.

---

## Frontend surfaces

Three surfaces, rebuilt around one scalar value instead of independent flags. Ships against the
backend's Phase 2 dual-write window — do not depend on Phase 3 or 4 to go live.

### Company panel — type editor row

`apps/admin/src/modules/mm-company-v2/presenters/components/organisms/company-detail-drawer/company-type-row.tsx` **(new)** — replaces the inline badge block in `company-details-panel.tsx`.

**States** — default (single badge + pencil, permitted roles only) · untyped (muted "Untyped" pill,
same edit control, not an error) · editing (single-select, all 8 values + an "Untyped" option mapped
to `null`, helper text below) · saving (optimistic badge swap, spinner on the pencil; revert +
inline error on failure) · read-only role (badge only).

**Permission gate** — verified in repo: `useAdminRoles().isMainRoleIn([AdminMainRole.SDR, AdminMainRole.KAM])`
→ pencil and dropdown render only when **not** in that lean set. Same gate `company-details-panel.tsx`
already uses for the `isPortfolio` row.

**Data read** — add `companyClassification?: CompanyType | null` to `CompanyDetailEntity`
(`company-detail-drawer/types.ts`, next to `isSponsor`/`isPortfolio`/`isCorporate` at lines 84–86),
and to the row type the list passes into the drawer.

**Labels** — one source for dropdown, chips and badge: `COMPANY_TYPE_OPTIONS` in
`adapters/constants.ts` (Sponsor · Portfolio · Bank · Advisory Firm · Law Firm · Accounting Firm ·
Corporate · Other). The synthetic `{ value: null, label: 'Untyped' }` entry is appended in the UI
layer only — `null` is never a real enum member.

**Behaviour change** — the old badge row could show more than one badge (e.g. Sponsor + Corporate).
Post-migration a company shows exactly one type or "Untyped": the visible face of the G6
priority-collapse decision. The first time an admin opens a formerly multi-typed company they see
only the winner. No extra copy proposed beyond the change-history entry — flag if the CPO wants an
inline callout.

### Bulk action bar — set company type

`apps/admin/src/modules/mm-company-v2/presenters/components/molecules/companies-bulk-action-bar/set-company-type-action.tsx` **(new)** — registered in `action-buttons.tsx` next to the bucket/labels actions.

Select rows or "all matching" → bulk bar shows "Set company type" → panel opens with a single-select
(8 values + Untyped), a count line, Confirm / Cancel → `bulkUpdateMmCompanyV2Fields` with the chosen
value → success toast + list refetch, rows update in place. Count-drift error shows the inline
message and keeps the panel open with a "Refresh count" action.

### List filter chips

`apps/admin/src/modules/mm-company-v2/presenters/.../map-company-filters-to-condition.ts`

```
// before: isSponsor: true, isCorporate: true (OR)
// after:
bCompanyTypeV2: { operator: "OR", items: ["Sponsor", "Bank"] }
// "Untyped" chip:
bCompanyTypeV2: { operator: "OR", items: [null] }
```

Condition-mapping **shrinks** to one field. Chips are multi-select (OR within the group); counts come
from the existing facet call, now faceting on one field instead of eight.

**Same field swap, also required:** `companies-list-template/index.tsx` (the three quick-filter keys
at ~line 926: `bIsSponsor` / `bIsPortfolio` / `bIsCorporate` → one `bCompanyTypeV2` key) and
`hooks/companies/build-company-quick-filters.ts` (same three legacy keys, ~line 58).
`company-status-tags/index.tsx` (row-level tag chips, `tag:isSponsor` etc.) is a separate, smaller
swap — **out of scope** for this pass unless PM wants row tags updated too.

---

## Completion checklist

* [ ] Industry-classification coverage reviewed with the CPO before the dry-run — confirm `Personnel` and other former-Agency signals correctly classify as `Other`.
* [ ] Dry-run backfill histogram attached to the G6 approval thread before any production write.
* [ ] Every `companyClassification` write (single edit, bulk edit, backfill) produces a change-history entry readable in the same UI as legacy-flag history.
* [ ] Provider refresh collapses correctly post-refresh — verified by refreshing a multi-flag company and reading the value back unchanged.
* [ ] Bulk edit on a stale "all matching" count still rejects with `MM_COMPANY_DIFFERENT_COUNT_QUANTITY`.
* [ ] Type editor visible only to permitted roles; read-only roles see the single badge, no pencil.
* [ ] A formerly multi-typed company shows exactly its collapsed winner post-Phase-2, with no layout break from the old multi-badge row.
* [ ] Filter chips: each of the 8 values + Untyped returns the right facet count; multi-select ORs correctly; the quick-filter row is rebuilt on the same field.
* [ ] Phase 4 (contract) has its own PR, its own review, and runs only after the ≥ 1-week dual-read monitoring window.

## Open questions

| # | Type | What is missing | Blocks | Who can answer |
|---|---|---|---|---|
| Q1 | Gate | G6 priority order — the collapse is only as right as this list | Phase 2 production backfill | CTO + CPO |
| Q2 | Gate | G4 taxonomy end-state (`ONE-WAY`) | Vocabulary consolidation (`QW-3`) | CTO |
| Q3 | `UNVERIFIED` | Which existing Select/Dropdown component to reuse for the single-select — not located this pass; check the design system's form components first | Panel editor build | Frontend lead |
| Q4 | `UNVERIFIED` | Whether `company-status-tags/index.tsx` row-level tags are in scope | Filter-chip work | PM |
| Q5 | Process | GraphQL codegen — confirm the shared `.gql.ts` fragments and inputs are regenerated after the backend enum ships, before wiring the mutations | Frontend wiring | Frontend lead |
| Q6 | `PRD GAP` | P13 — the code derives Corporate as a complement; the product ruling says "possible client". The code is today's reality until `QW-1` lands | Corporate-filtered reporting | CPO (owns the G5 mapping) |
