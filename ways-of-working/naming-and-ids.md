# Naming, IDs and status

## Activity IDs

IDs come from the matrix and never change: `<stage>.<sequence>` — `0.01`, `4.05`, `10.04`.
They are the join key between the spreadsheet, this documentation, tickets, and branch names.
A retired activity keeps its ID and is marked `Retired`; the number is never reused.

## File names

| Thing | Pattern | Example |
|---|---|---|
| Stage folder | `kebab-stage-name` — the Revenue Stage name, no number | `contracting` |
| Activity folder | `<id>-<kebab-slug>/` | `0.01-company-type/` |
| Stakeholder page | fixed — `README.md`, `for-the-ceo.md`, `for-operations.md`, `for-product.md`, `for-the-cto.md`, `for-the-tech-team.md`, `for-developers.md` | never renamed, never extended |
| Runbook | `runbook.md` inside the activity folder | `0.04-…/runbook.md` |
| Platform page | `kebab-topic.md` | `duplicate-detection.md` |

Slugs are lowercase, hyphenated, ASCII, ≤60 chars. Shorten the matrix wording rather than
inventing a new name — keep the first meaningful words.

## Status vocabulary

Put one of these in the page front block. Nothing else counts as a status.

| Status | Meaning |
|---|---|
| `Draft` | Written, not yet reviewed by the activity owner |
| `In review` | Circulated; the 48h silence clock is running |
| `Approved` | Decisions taken; safe to build from |
| `Superseded` | Replaced — the front block links to the successor |
| `Retired` | Activity no longer part of the engine |

## Pass labels

`Pass 1` = `README.md` + `for-the-ceo.md` + `for-operations.md` + quick-win cards in `for-developers.md`.
`Pass 2` = `for-the-cto.md` + the rest of `for-developers.md`.
On demand = `for-product.md`, `for-the-tech-team.md`, the known-error table in `for-operations.md`.

## Role abbreviations

`SDR` · `KAM` (key account manager) · `Junior Ops` · `Senior Ops` · `Automation`.
`R` = primary owner, `S` = supporting role, blank = no routine responsibility.
Short term and mid term are separate columns — always state which one you mean.

## Cross-references

Link by activity ID, not by title: `` [`0.02`](../data-foundation/0.02-…/README.md) ``.
Titles get edited; IDs do not.
