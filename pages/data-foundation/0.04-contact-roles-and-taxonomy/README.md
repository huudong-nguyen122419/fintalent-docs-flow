# `0.04` Refresh & update contacts, roles and decision-maker taxonomy

**Activity** `0.04` — "Refresh & Update, Identify and define contact roles and decision-maker taxonomy"
**Stage** 0. Data Foundation · **Owner** short term SDR `R`, Senior Ops `S` · mid term SDR `S`, KAM `S`, Senior Ops `R`, Automation `S`
**Automation target** AI-assisted classification
**Status** `Draft` · **Pass** — script deliverable only · **Last verified** 2026-08-12 on `fintalent-backend-microservices@develop`
**Source** `working-docs/data-foundation/0.04 Script - Import Contacts from CSV.dc.html` (rev 1) · **Synced** 2026-08-12

> **Scope note from the matrix:** this activity defines contact types. Selecting the actual person
> to approach remains part of Prospecting (`1.05`).

## What exists today

One deliverable: a resumable bulk-import script that turns a CSV of LinkedIn URLs into enriched,
company-linked, dedupe-safe contacts.

**CSV → enrich → contact → link company → dedupe.** 10 000 rows ≈ 10 days at the 1 000/day
enrichment cap. Existing contacts are never touched — their ids are exported for a separate refresh
pass.

→ [Runbook](runbook.md) — the five steps, the commands, the flags, and what to confirm before an
apply run.

## Not yet documented

The taxonomy itself — the standardized functions, seniority levels and buying-process roles used to
classify and prioritize contacts. `UNVERIFIED — no source read for the role taxonomy; only the
import path has been traced.`
