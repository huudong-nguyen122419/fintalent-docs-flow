# 3 · Change briefs

> part of [0.01 · Define & maintain company type](README.md)

**For: CTO.** _Change contracts, not implementations. Every change is tagged <mark style="color:$success;background-color:$success;">DECIDED</mark> or <mark style="color:$warning;background-color:$warning;">NEEDS CTO</mark> and traced to a pain ID. JSON samples live only here. Exact file evidence: technical appendix (rev 3)._

{% hint style="info" %}
**Takeaway:** four changes make company type mean the right thing, writable, audited and single-vocabulary; one call remains yours — the taxonomy end-state (G4), now gating quick-win card 3; the industry → type mapping (G5) defaults in 48h.
{% endhint %}

## 3A · Backend change brief

**Current status:** one company store (setting service) holds three independent yes/no type values plus parallel vocabularies (legacy text label, automated tags, buckets, labels). Writes arrive through a single-record edit (whitelisted fields, audited), a bulk edit (count-guarded, unaudited), and a provider-refresh job (overwrites everything, derives "Corporate" as the complement of the other two flags, manual trigger disconnected). Every write already re-scores company data health automatically.

### 3A-B · Database updates

| Store           | Change needed                                                                                                                                                                                                                                                                                                         | Why      | Migration risk                                                                                                                                                                 | Decision                                                              |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| Company records | No schema change for editing — the three type values already exist; only the write whitelist widens.                                                                                                                                                                                                                  | P1 · G1  | None.                                                                                                                                                                          | <mark style="color:$success;background-color:$success;">DECIDED</mark>                                                             |
| Audit log store | No new table — bulk writes reuse the existing per-field change-history records the single edit already produces.                                                                                                                                                                                                      | P6       | Low — write volume rises on bulk actions; events are batched.                                                                                                                  | <mark style="color:$success;background-color:$success;">DECIDED</mark>                                                             |
| Company records | Add five new type values — **Advisory Firm, Law Firm, Banks, Accounting Firm, Other** — following the existing independent-flag pattern; the provider's industry evidence (sector, industry hierarchy, NAICS codes) already stored per company drives them; a **re-runnable backfill** reclassifies existing records. | P13 · G5 | Low–medium — additive flags, deterministic from stored fields; the search index gains the new fields (one rebuild window needed); a re-run applies an amended mapping cleanly. | <mark style="color:$success;background-color:$success;">DECIDED</mark> mechanics; the industry → type mapping is G5 (48h default). |

### 3A-C · API updates

**Edit one company** · existing operation, widened · <mark style="color:$success;background-color:$success;">DECIDED · G1</mark>

Accepts all type values — the existing three plus Advisory Firm / Law Firm / Banks / Accounting Firm / Other (`true` / `false` / `null-to-clear`) — and writes the audit record — resolves P1. With refresh protection descoped (P3), the next provider refresh may overwrite a manual value. Back-compat: additive input fields, non-breaking, no version bump.

{% columns %}
{% column %}
```graphql
# request
mutation updateMmCompanyV2Fields
{ "id": "663d…9f1",
  "input": {
    "isSponsor": true,
    "isCorporate": false } }
```
{% endcolumn %}

{% column %}
```json
// success
{ "data": { "updateMmCompanyV2Fields": {
    "id": "663d…9f1", "isSponsor": true,
    "isPortfolio": false, "isCorporate": false } } }

// error (validation)
{ "errors": [ { "message": "isSponsor must be a boolean",
    "extensions": { "code": "BAD_USER_INPUT" } } ] }
```
{% endcolumn %}
{% endcolumns %}

**Bulk edit companies** · existing operation, widened + audited · <mark style="color:$success;background-color:$success;">DECIDED</mark>

Gains the same three type fields and emits the per-company audit records the single edit already produces — resolves P6, enables bulk type assignment. The count-guard rejection stays; its message gains recovery guidance (P12, copy in 3B-c). Back-compat: additive, non-breaking.

{% columns %}
{% column %}
```graphql
# request
mutation bulkUpdateMmCompanyV2Fields
{ "condition": { "ids": ["663d…9f1", "663d…a02"] },
  "input": { "isCorporate": true } }
```
{% endcolumn %}

{% column %}
```json
// success
{ "data": { "bulkUpdateMmCompanyV2Fields": true } }

// error (count drift on "all matching")
{ "errors": [ { "message": "count mismatch",
    "extensions": { "code": "MM_COMPANY_DIFFERENT_COUNT_QUANTITY" } } ] }
```
{% endcolumn %}
{% endcolumns %}

### 3A-D · Decision gates

| Gate   | Rule / question                                                                                               | Options considered                                                                                                                                                                                                                                                                                                                          | Status                                                                                                                                                                                                                                                                                                  |
| ------ | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **G1** | Who may edit company type; are the three values exclusive?                                                    | a) all admins, single-select · b) senior data roles, independent values                                                                                                                                                                                                                                                                     | <mark style="color:$success;background-color:$success;">DECIDED</mark> — **(b)**: matches stored data and the existing role model; accepted via the [§0](decision-strip.md) strip.                                                                                                                                                                                                        |
| **G2** | Do manual type fixes survive provider refreshes?                                                              | a) provider always wins · b) admin-owned values protected via provenance                                                                                                                                                                                                                                                                    | <mark style="color:$info;background-color:$info;">DESCOPED</mark> — product removed the refresh-protection work on 2026-08-11; today's behaviour stays (a) provider always wins. P3 remains on the pain register.                                                                                                                                              |
| **G3** | Give untyped companies a visible home + backfill?                                                             | a) leave invisible · b) untyped filter now, classifier backfill after                                                                                                                                                                                                                                                                       | <mark style="color:$info;background-color:$info;">WITHDRAWN</mark> — untyped-at-birth accepted as intended behaviour (2026-08-11); no untyped filter or classifier backfill ships.                                                                                                                                                                             |
| **G5** | Corporate = client-eligible only (PRD gap P13): which provider industries map to which of the five new types? | a) draft mapping on the provider's sector/industry codes — commercial & investment banks → Banks; M\&A / corporate-finance advisories, consultancies → Advisory Firm; accountancy & audit → Accounting Firm; law firms → Law Firm; other non-client service industries → Other · b) coarser split — Banks plus one combined "Services" type | <mark style="color:$warning;background-color:$warning;">DEFAULTS 48H</mark> — auditor recommends (a), reviewed quarterly; REVERSIBLE via the re-runnable backfill. Reclassified firms leave the Corporate filter by design; unmapped industries stay Corporate. Companies with no industry data stay Corporate and are logged for review. CPO owns the mapping ([§0](decision-strip.md)). |
| **G4** | Taxonomy end-state: which vocabulary is canonical for "company type"?                                         | a) promote the official type tag (now eight values), retire the legacy text label · b) adopt the automated two-level classification as canonical                                                                                                                                                                                            | **<mark style="color:$warning;background-color:$warning;">NEEDS CTO</mark>** — auditor recommends (a) now, revisit (b) once a review queue exists (P11). Resolves P5; gates quick-win card 3.                                                                                                                                                                        |

### 3A-E · Completion checklist _(verification only)_

* Setting a type value via single edit is visible on re-read, in the list row, and in the change history (P1).
* A bulk edit of N companies produces N history entries identical in shape to single-edit entries (P6).
* After the backfill, a provider company in a mapped service industry (e.g. an M\&A advisory) is tagged Advisory Firm and absent from the Corporate filter, while a client-industry peer keeps its Corporate tag (P13).
* Backend coverage modes each covered-by-test or explicitly waived: validation rejection, authz rejection, dependency failure, timeout/retry, double submit, partial failure.
* Contract map (rev 3, 7c) re-run: rows 1 and 2 clear; no new gaps; no new swallowed errors introduced.

## 3B · Frontend change brief

**Current status:** the companies list and company panel show type as read-only badges; admins segment with buckets/labels instead; a finished refresh button exists but was never placed on a screen; there is no view of untyped companies and no explanation when a panel opens on a merged/removed record.

### 3B-B · API surface for the frontend

| Operation                | Status       | What the frontend must change                                                                                                                               | Replaces                                                                                                                         | Pain     |
| ------------------------ | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------- |
| Edit one company         | <mark style="color:$info;background-color:$info;">CHANGED</mark>    | Add a role-gated "Company type" editor row in the company panel (same inline pattern as lifecycle); optimistic badge update; extend the editable-field set. | Read-only badges as the only type surface.                                                                                       | P1       |
| Bulk edit companies      | <mark style="color:$info;background-color:$info;">CHANGED</mark>    | Add "Set company type" to the bulk action bar (reuses the bucket/labels panel pattern); improve the count-drift rejection message (copy below).             | Bucket/labels as the only bulk taxonomy.                                                                                         | P6 · P12 |
| List filter              | <mark style="color:$info;background-color:$info;">CHANGED</mark>    | Add "Advisory Firm", "Law Firm", "Banks", "Accounting Firm" and "Other" values to the company-type filter chips; counts come from the existing facet call.  | —                                                                                                                                | P13      |
| Legacy text type display | <mark style="color:$info;background-color:$info;">DEPRECATED</mark> | Stop presenting the legacy free-text type on legacy surfaces once G4 is answered.                                                                           | Replaced by the official type tag everywhere. Removal note: hide after G4 decision; field stays in storage until pass-3 cleanup. | P5       |

### 3B-C · Changed journey + UX copy

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

### 3B-D · Completion checklist _(verification only)_

* Type editor visible only to permitted roles; read-only roles see badges unchanged.
* Each changed screen covers happy / empty / loading / error states.
* Full journey walk: list → filter → panel → edit type → badge + history reflect it.
* Deep-link, back and refresh keep filter state; concurrent-edit pass shows the improved rejection message.
