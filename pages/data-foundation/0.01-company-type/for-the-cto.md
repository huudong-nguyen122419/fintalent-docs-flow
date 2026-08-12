# For the CTO — `0.01` Define & maintain company type

**Four changes make company type mean the right thing, writable, audited and single-vocabulary. One
call remains yours — the taxonomy end-state (G4), which gates quick-win card 3. The industry → type
mapping (G5) defaults in 48h.**

Change contracts, not implementations. Every change is tagged `DECIDED` or `NEEDS CTO` and traced to
a pain ID. JSON samples live only here.

Scope rulings baked in: refresh-protection (P3) and on-demand-refresh (P2) descoped;
untyped-at-birth (P4) accepted as intended behaviour.

## Gates

| Gate | Rule / question | Options considered | Status |
|---|---|---|---|
| **G4** | Taxonomy end-state: which vocabulary is canonical for "company type"? | a) promote the official type tag (now eight values), retire the legacy text label · b) adopt the automated two-level classification as canonical | **`NEEDS CTO`** · `ONE-WAY` — auditor recommends (a) now, revisit (b) once a review queue exists (P11). Resolves P5; gates quick-win card 3 |
| **G5** | Corporate = client-eligible only (`PRD GAP` P13): which provider industries map to which of the five new types? | a) draft mapping on the provider's sector/industry codes — commercial & investment banks → Banks; M&A / corporate-finance advisories and consultancies → Advisory Firm; accountancy & audit → Accounting Firm; law firms → Law Firm; other non-client service industries → Other · b) coarser split — Banks plus one combined "Services" type | **DEFAULTS IN 48H** — auditor recommends (a), reviewed quarterly; `REVERSIBLE` via the re-runnable backfill. Reclassified firms leave the Corporate filter by design; unmapped industries stay Corporate. Companies with no industry data stay Corporate and are logged for review. CPO owns the mapping |
| **G6** | Priority order used to collapse today's independent flags into one value | see below | **OPEN** — CTO/CPO confirm; the backfill is only as right as this list |
| **G1** | Who may edit company type; are the three values exclusive? | a) all admins, single-select · b) senior data roles, independent values | **`DECIDED`** — (b): matches stored data and the existing role model |
| **G2** | Do manual type fixes survive provider refreshes? | a) provider always wins · b) admin-owned values protected via provenance | **DESCOPED** 2026-08-11 — today's behaviour stays (a). P3 remains on the pain register |
| **G3** | Give untyped companies a visible home + backfill? | a) leave invisible · b) untyped filter now, classifier backfill after | **WITHDRAWN** — untyped-at-birth accepted as intended behaviour |

---

## Backend

**Where it stands** One company store (setting service) holds three independent yes/no type values
plus parallel vocabularies — legacy text label, automated tags, buckets, labels. Writes arrive
through a single-record edit (whitelisted fields, audited), a bulk edit (count-guarded, unaudited),
and a provider-refresh job (overwrites everything, derives "Corporate" as the complement of the
other two flags, manual trigger disconnected). Every write already re-scores company data health.

### Database updates

| Store | Change needed | Why | Migration risk | Decision |
|---|---|---|---|---|
| Company records | No schema change for editing — the three type values already exist; only the write whitelist widens | P1 · G1 | None | `DECIDED` |
| Audit log store | No new table — bulk writes reuse the existing per-field change-history records the single edit already produces | P6 | Low — write volume rises on bulk actions; events are batched | `DECIDED` |
| Company records | Add five new type values — Advisory Firm, Law Firm, Banks, Accounting Firm, Other — following the existing independent-flag pattern; the provider's industry evidence (sector, industry hierarchy, NAICS codes) already stored per company drives them; a re-runnable backfill reclassifies existing records | P13 · G5 | Low–medium — additive flags, deterministic from stored fields; the search index gains the new fields (one rebuild window); a re-run applies an amended mapping cleanly | `DECIDED` mechanics; the mapping is G5 (48h default) |

### API updates

#### Edit one company — existing operation, widened · `DECIDED` · G1

Accepts `companyType` — one of the eight enum values, or `null` to clear — and writes the audit
record; resolves P1. Setting the type is a straight replace: no priority logic runs on manual edits
(priority only collapses the legacy flags at backfill). With refresh protection descoped (P3), the
next provider refresh may overwrite a manual value. **Back-compat:** additive input field; the
deprecated booleans stay accepted through the dual-write window.

```json
// request — mutation updateMmCompanyV2Fields
{ "id": "663d…9f1",
  "input": { "companyType": "LawFirm" } }

// success
{ "data": { "updateMmCompanyV2Fields": { "id": "663d…9f1", "companyType": "LawFirm" } } }

// error (validation)
{ "errors": [ { "message": "Value \"Law\" does not exist in \"CompanyType\"",
                "extensions": { "code": "BAD_USER_INPUT" } } ] }
```

#### Bulk edit companies — existing operation, widened + audited · `DECIDED`

Gains the same `companyType` field and emits the per-company audit records the single edit already
produces — resolves P6, enables bulk type assignment. One value is applied to every matched company
(replace, not merge — there is nothing to merge on a scalar). The count-guard rejection stays; its
message gains recovery guidance (P12). **Back-compat:** additive, non-breaking.

```json
// request — mutation bulkUpdateMmCompanyV2Fields
{ "condition": { "ids": ["663d…9f1", "663d…a02"] },
  "input": { "companyType": "Corporate" } }

// success
{ "data": { "bulkUpdateMmCompanyV2Fields": true } }

// error (count drift on "all matching")
{ "errors": [ { "message": "count mismatch",
                "extensions": { "code": "MM_COMPANY_DIFFERENT_COUNT_QUANTITY" } } ] }
```

#### Migrate to a single type field — new `companyType` enum · `DECIDED` shape · G6 priority order open

One enum replaces the per-type booleans (`isSponsor` / `isPortfolio` / `isCorporate` plus the five
new service-firm flags). Values are additive later, so the list starts narrow.

| Enum value | Label (UI) | Means | Backfilled from |
|---|---|---|---|
| `Sponsor` | Sponsor | PE / VC / investment firm | `isSponsor` |
| `Portfolio` | Portfolio | Sponsor-backed company | `isPortfolio` |
| `Corporate` | Corporate | Operating company, not sponsor-backed | `isCorporate` |
| `AdvisoryFirm` | Advisory Firm | M&A / strategy / consulting | G5 industry mapping |
| `LawFirm` | Law Firm | Legal counsel | G5 industry mapping |
| `Bank` | Bank | Investment / commercial bank | G5 industry mapping |
| `AccountingFirm` | Accounting Firm | Audit / tax / transaction services | G5 industry mapping |
| `Other` | Other | Typed, but none of the above — distinct from untyped. Also covers what would have been PA (placement agents) and Agency (executive search / staffing), both withdrawn as distinct types | G5 fallback |

**Shape** (decided by product, 2026-08-11): `companyType: CompanyType` — a single scalar, not an
array. Today's flags are independent, so multi-typed records are collapsed by a fixed priority order
— highest type wins (Portfolio + Corporate → Portfolio). No flag at all → `null` = untyped, per the
P4 ruling; `Other` means "typed, but none of these". Naming: PascalCase, singular nouns (Banks →
Bank). PA and Agency were withdrawn; both classify as `Other`.

**Priority order (G6 — CTO/CPO confirm):**

```
Sponsor > Portfolio > Bank > AdvisoryFirm > LawFirm > AccountingFirm > Corporate > Other
```

Capital-side roles rank above service roles, and Corporate ranks last-but-one because it is derived
today as the complement of the other flags — a weak signal that should never beat a positive one.
Lossy by design: the discarded values are written once to the change-history record at backfill, so
nothing is unrecoverable.

```json
// request — same mutation, new field
{ "id": "663d…9f1", "input": { "companyType": "Portfolio" } }

// clear the type (untyped)
{ "id": "663d…9f1", "input": { "companyType": null } }

// success — dual-write window returns both shapes
{ "data": { "updateMmCompanyV2Fields": {
    "id": "663d…9f1",
    "companyType": "Portfolio",   // was Portfolio + Corporate
    "isPortfolio": true,           // deprecated
    "isCorporate": false } } }     // lowered by priority

// error (unknown value — PA/Agency no longer exist)
{ "errors": [ { "message": "Value \"PA\" does not exist in \"CompanyType\"",
                "extensions": { "code": "BAD_USER_INPUT" } } ] }
```

### Done when

* [ ] Setting a type value via single edit is visible on re-read, in the list row, and in the change history (P1).
* [ ] A bulk edit of N companies produces N history entries identical in shape to single-edit entries (P6).
* [ ] After the backfill, a provider company in a mapped service industry is tagged Advisory Firm and absent from the Corporate filter, while a client-industry peer keeps its Corporate tag (P13).
* [ ] Backend coverage modes each covered-by-test or explicitly waived: validation rejection, authz rejection, dependency failure, timeout/retry, double submit, partial failure.
* [ ] Contract map re-run: rows 1 and 2 clear; no new gaps; no new swallowed errors introduced.

---

## Frontend

**Where it stands** The companies list and company panel show type as read-only badges; admins
segment with buckets and labels instead; a finished refresh button exists but was never placed on a
screen; there is no view of untyped companies and no explanation when a panel opens on a merged or
removed record.

### Surface changes

| Operation | Status | What the frontend must change | Replaces | Pain |
|---|---|---|---|---|
| Edit one company | `CHANGED` | Add a role-gated "Company type" editor row in the company panel (same inline pattern as lifecycle); optimistic badge update; extend the editable-field set | Read-only badges as the only type surface | P1 |
| Bulk edit companies | `CHANGED` | Add "Set company type" to the bulk action bar (reuses the bucket/labels panel pattern); improve the count-drift rejection message | Buckets/labels as the only bulk taxonomy | P6, P12 |
| List filter | `CHANGED` | Add "Advisory Firm", "Law Firm", "Banks", "Accounting Firm" and "Other" values to the company-type filter chips; counts come from the existing facet call | — | P13 |
| Legacy text type display | `DEPRECATED` | Stop presenting the legacy free-text type on legacy surfaces once G4 is answered. Hide after the G4 decision; the field stays in storage until pass-3 cleanup | Replaced by the official type tag everywhere | P5 |

### Changed journey

1. **Open the company panel** — the type row now shows badges plus an Edit control (permitted roles).
2. **Change the type** — pick from all eight types; save inline; badges update instantly.
3. **Recorded** — the change lands in the company's history with the admin's name.

### Proposed copy

| Surface | Current copy | Proposed copy | State | Decision |
|---|---|---|---|---|
| Company panel — type row | (badges only, no label or control) | **"Company type"** · helper: *"May be overwritten by the next provider refresh."* | Default / edit | `DECIDED` |
| Bulk "all matching" rejection | Raw error toast surfacing the server message (`UNVERIFIED` verbatim) | *"The matching list changed while you were editing. Review the updated count and try again."* | Error toast | `DECIDED` |
| Type filter chips | "Sponsor · Portfolio · Corporate" | Add chips: *"Advisory Firm · Law Firm · Banks · Accounting Firm · Other"* | Filter bar | `DECIDED` |
| Company panel — missing record | (empty panel, no message) | *"This company was merged or removed. Open the surviving record from its parent link."* | Empty state | `DECIDED` |

### Done when

* [ ] Type editor visible only to permitted roles; read-only roles see badges unchanged.
* [ ] Each changed screen covers happy / empty / loading / error states.
* [ ] Full journey walk: list → filter → panel → edit type → badge and history reflect it.
* [ ] Deep-link, back and refresh keep filter state; concurrent-edit pass shows the improved rejection message.

---

## Sequencing & rollout

Cards 1, 2 and 4 can start in parallel once G5's 48h window closes; card 3 alone waits on the G4
answer.

| Order | Change | Gate | Effort | Notes |
|---|---|---|---|---|
| 1 | Reclassify non-clients into the five new types (`QW-1`) | G5 — defaults in 48h | E3 ≈ 2–3 days | Schema + enum flags → mapping constant → derivation → re-runnable backfill → index + filter chips. Staging before/after counts per type reviewed with the CPO. One index rebuild window — schedule with the CTO |
| 2 | Make company type editable (`QW-2`) | G1 — decided | E2 ≈ 1–3 days | Parallel to order 1. With P3 descoped, a manual value can be overwritten by the next scheduled refresh — the panel helper copy warns stewards |
| 3 | Audit bulk edits (`QW-4`) | decided | E2 ≈ 1–2 days | Independent. One batched history event per bulk action, same shape as single-edit entries |
| 4 | Consolidate the type vocabularies (`QW-3`) | G4 — `NEEDS CTO` · `ONE-WAY` | E4 ≈ 3–5 days | Starts only after G4 is signed and card 1 has landed the eight-value tag. Hide, don't delete, the legacy free-text field |

### Rollout guardrails

* The backfill is re-runnable and reversible: an amended G5 mapping re-applies cleanly; batch writes emit one batched health-recompute event, mirroring the existing recompute migrations.
* Companies with no industry data stay Corporate and are logged for review — never silently moved to `Other`.
* The derivation reruns on every provider refresh; with refresh protection descoped, the mapping is the only durable control — manual reclassifications do not survive.
* CPO owns the industry → type mapping and reviews it quarterly.
