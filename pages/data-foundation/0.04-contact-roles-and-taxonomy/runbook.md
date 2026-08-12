# Runbook — bulk import contacts from a LinkedIn-URL CSV

**Activity** `0.04` · **Runs on** production only · **Owner** Senior Ops / backend
**Script** `migrations-v2/contact/contact.import-from-csv.v2.ts` (repo: `fintalent-backend-microservices`, branch `develop`)
**Last verified** 2026-08-12

## What this does

Takes a CSV — or a JSON array like the uploaded sample — whose only required column is
`linkedinUrl`, and **inserts new contacts only**. Every run is checkpointed to a state file, so the
same command resumes after the daily enrichment budget, a crash, or Ctrl-C.

`STAGE=production` is required: the enrichment provider refuses local and staging by design.
Dry-run is the default and writes nothing — but it still spends enrichment calls, so always start
with `--limit=10`.

## The five steps, and what each reuses

No new matching or normalization logic was invented; each step reuses the platform's own building
block.

| Step | What the script does | Reused from the repo |
|---|---|---|
| 1 · Read the CSV (≤10k) | Auto-detects the `linkedinUrl` column and delimiter, handles quoted cells; accepts a `.json` array. Normalizes every URL to the canonical dedup key; drops invalid URLs and in-file duplicates with counts. Hard cap 10 000. | `normalizeLinkedinUrlV2` (`libs/core`) — the exact function `createContactV2` uses for `dedupLinkedinKey` |
| 2 · Enrich in batches | 2 concurrent calls, 30s timeout each, ~400ms between batches. A per-UTC-day budget (default 1 000) stops the run cleanly; re-running the identical command resumes from the state file. Failures are recorded per row and retryable. | `enrichWithProxycurlData` (`libs/provider` proxy-curl) — raw response auto-archived to S3 · `BULK_ENRICH_CONCURRENCY=2` · `ENRICHLAYER_DAILY_CAP=1000` (`enrich-guards.ts`) |
| 3 · Profile → contact | The provider returns a contact-shaped profile (names, avatar, title, address, emails, educations, work experiences, total years of experience). The script adds the v2 identity trio — `linkedinUrl` / `normalizedLinkedinUrl` / `dedupLinkedinKey` — plus `emailsLower`, `isActive`/`isDeleted`, `isSecondary` on work experiences, and audit stamps (`proxycurlCrawledAt`, `lastLinkedinEnrichment`, `importSource` = file name). | `fromProxyCurlToContact` (inside the provider) · `applyWorkExperienceIsSecondary` (`libs/core`) |
| 4 · Fill company and lifecycle | Per batch, pulls candidate `mm_companies` by normalized company LinkedIn URL + `legalName`/`aliasNames` (case-insensitive collation) and `email_validations` for the email pool, then links `linkedCompanies`, per-work-experience emails, `hasWorkEmail`, `countPortfolios`, `emailDomains`, `isSponsor`/`isPortfolio`/`isCorporate`. Defaults: `userTypes=[Prospect]`, `lifecycle=Prospect`, `sourceType=salesQL` — all overridable. `--skip-link` defers linking to the standalone pass. | `linkContactToCompanies` + `buildContactLinkUpdate` (`libs/utils`) — same matcher as `MicroLinkContactToCompaniesUC` · `checkHasWorkEmail` / `getPortfolioIds` / `typeEmailValidation` (`libs/utils`) |
| 5 · Cleanup / merge duplicates | Duplicates never enter: in-file dupes dropped at parse; a DB pre-check on `dedupLinkedinKey` / `normalizedLinkedinUrl` / `linkedinUrl` skips existing contacts and exports their ids to `<state>.existing.csv`; each insert re-checks the key right before writing (guards parallel runs). | Same identity keys as `MicroContactFindExistingByIdentityUC` · post-run: `contact.merge-duplicates.v2.ts` + `contact.scan-data-health.v2.ts` |

## Steps

```bash
# 1 · dry-run 10 rows — verify parsing, dedup counts, sample contacts
STAGE=production npx ts-node -r dotenv/config \
  migrations-v2/contact/contact.import-from-csv.v2.ts --file=./import/batch-1.csv --limit=10

# 2 · apply — resumable; re-run the SAME command after the daily cap or a crash
STAGE=production npx ts-node -r dotenv/config \
  migrations-v2/contact/contact.import-from-csv.v2.ts --file=./import/batch-1.csv --apply

# 3 · retry rows that failed enrichment
STAGE=production npx ts-node -r dotenv/config \
  migrations-v2/contact/contact.import-from-csv.v2.ts --file=./import/batch-1.csv --apply --retry-failed

# 4 · finish: refresh the skipped existing contacts, then dedupe + health passes
#     existing ids are in ./import/batch-1.csv.state.json.existing.csv
migrations-v2/contact.renew-proxycurl-data.v2.ts --ids=<existing ids>
migrations-v2/contact/contact.merge-duplicates.v2.ts          # dry-run first
migrations-v2/contact/data-health/contact.scan-data-health.v2.ts
```

## Flags

| Flag | Default | Meaning |
|---|---|---|
| `--file=` | required | `.csv` (column auto-detected) or `.json` array of objects with `linkedinUrl` |
| `--apply` | off (dry-run) | Actually insert contacts |
| `--limit=` | ∞ | Rows this run |
| `--concurrency=` | 2 | Parallel enrichment calls |
| `--daily-cap=` | 1000 | Max calls per UTC day |
| `--state=` | `<file>.state.json` | Checkpoint file: per-row status + daily budget counter — delete it to start over |
| `--lifecycle=` / `--user-type=` | `Prospect` / `Prospect` | Classification stamped on every new contact (`lifecycle` + `lifeCycles`, `userTypes`) |
| `--skip-link` | off | Insert without company linking; run `contact.link-to-companies.v2.ts` later |
| `--retry-failed` | off | Re-attempt rows marked `ENRICH_FAILED` in the state file |

## Verify

Row state machine, as recorded in the state file:

| From | To | Meaning |
|---|---|---|
| `PENDING` | `EXISTS_IN_DB` | Skipped; id exported to `<state>.existing.csv` |
| `PENDING` | `ENRICH_FAILED` | Retryable with `--retry-failed` |
| `PENDING` | `ENRICH_EMPTY` | No name and no work experience — not inserted |
| `PENDING` | `READY_DRY_RUN` | Dry-run only |
| `PENDING` | `INSERTED` | Written, with `contactId` |

## Confirm before the apply run

1. `lifecycle=Prospect`, `userTypes=[Prospect]` — matches the "no engagement yet" bucket in the
   lifecycle-recompute rules. Change per campaign if needed.
2. Existing contacts are **skipped, never merged** — refresh them with the renew script instead.
3. `requiredReview` is not set. Add it if imported contacts should enter the review queue.
4. Sales-Navigator-style URN URLs are passed to the provider as-is; its response canonicalizes the
   public identifier.

**Resumability** — the state file makes every run idempotent: re-running the identical command
picks up where the budget, crash or interrupt left off. Delete the state file to start over.
