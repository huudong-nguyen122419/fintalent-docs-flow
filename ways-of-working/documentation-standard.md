# Documentation standard

**One activity = one folder = seven pages, one per stakeholder.** The Revenue Engine Activity
Matrix defines the unit of work; the docs tree mirrors it exactly. If something does not map to an
activity ID, it belongs in `platform/`, not in `pages/`.

## Where a page goes

```
docs/
├── README.md                      GitBook landing
├── SUMMARY.md                     GitBook table of contents — a page not listed is not published
├── ways-of-working/               how we write docs
├── templates/
│   ├── activity/                  the seven stakeholder pages — copy this whole folder
│   └── runbook.md
├── pages/
│   ├── activity-matrix.md         mirror of the v0.9 spreadsheet
│   ├── data-foundation/
│   │   ├── README.md              stage index, derived from the matrix
│   │   └── 0.01-company-type/
│   │       ├── README.md               Decisions — everyone
│   │       ├── for-the-ceo.md          CEO
│   │       ├── for-operations.md       COO / Ops
│   │       ├── for-product.md          CPO / Product
│   │       ├── for-the-cto.md          CTO
│   │       ├── for-the-tech-team.md    Tech team
│   │       ├── for-developers.md       Junior / mid dev
│   │       └── runbook.md              only if the activity has a procedure to run
│   ├── prospecting/ … expansion/
└── platform/                      cross-cutting: data model, integrations, runbooks, environments
```

Stage folders are named after the **Revenue Stage** column in the matrix — no number prefixes.
Reading order is set by `SUMMARY.md`, not by the filesystem.

## The same seven pages, everywhere

Every activity folder uses the same file names, so anyone learns the layout once and can then open
their page in any activity without looking.

| File | Who owns it | What it answers | Produced in |
|---|---|---|---|
| `README.md` | Everyone | The one decision each stakeholder owes, and what happens on silence | Pass 1 |
| `for-the-ceo.md` | CEO | What it costs, what it is worth, which gates need a call | Pass 1 |
| `for-operations.md` | COO / Ops | What users report, who it hits, the workaround today | Pass 1 |
| `for-developers.md` | Dev | Files, steps, acceptance — quick wins first | Pass 1, extended in pass 2 |
| `for-the-cto.md` | CTO | What changes on each side; the gates needing a decision | Pass 2 |
| `for-product.md` | CPO | Scenarios, journeys, verbatim copy, pain register | On demand |
| `for-the-tech-team.md` | Tech team | Surfaces, contracts, gaps, open questions | On demand |

A page that has not been produced yet simply does not exist. Never create an empty file, and never
invent an eighth page — if content does not fit one of the seven, it is not part of the activity doc.

## What every page carries at the top

The activity `README.md` opens with the front block; the other six inherit its context by living in
the same folder.

* **Activity** — `ID` + name, verbatim from the matrix
* **Stage** and **owner** (short term / mid term), verbatim from the matrix
* **Status** — see [Naming, IDs and status](naming-and-ids.md)
* **Pass** — 1, 2, or which on-demand pages exist
* **Last verified** — ISO date + the branch the code was read on

## Language rules (inherited from the master prompt)

* Only `for-the-tech-team.md` and the fix cards in `for-developers.md` carry file paths, handlers or
  endpoints. Every other page reads cleanly to a non-engineer.
* Tables over prose. No paragraph over three sentences.
* Pain points are written as: journey moment → what they saw → what really happened.
* Never invent identifiers. Unverifiable behaviour is marked `UNVERIFIED — <what is missing>`.

## Keeping a page in sync

Each activity folder is the published half of a 1-to-1 pair with its working document in
`working-docs/<revenue-stage>/`. The working document is where revisions happen; the published pages
are where decisions are read.

| | `working-docs/` | `docs/pages/` |
|---|---|---|
| Holds | The interactive analysis, all revisions | The settled record, one page per stakeholder |
| Changes | Freely, no approval | Only on approval, only as a delta |
| Versioned by | rev number + `archive/` | The `Source` / `Synced` line in the activity `README.md` |

The activity `README.md` front block therefore carries two extra fields:

```
**Source** working-docs/data-foundation/0.01 Company Type Flow Analysis.dc.html (rev 5)
**Synced** 2026-08-12
```

If the working document has moved on, that line is the honest signal — bump it only when the pages
have actually been updated.

## Adding an activity

1. Pick the activity ID from the matrix.
2. **Ask for approval before saving anything into `docs/`.** No exceptions — see `CLAUDE.md`.
3. Copy `templates/activity/` to `pages/<stage>/<id>-<slug>/`, keeping the file names.
4. Add the folder to `SUMMARY.md` in the same commit, or GitBook will not publish it.
