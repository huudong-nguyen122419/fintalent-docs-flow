# Flow Audit

Point it at one user flow. It reads both sides of the code, documents the flow as scenarios, ranks every problem it finds by return on effort, and hands back a change brief for backend and one for frontend.

## Versions

| Version | Status | What changed |
|---|---|---|
| [v2](v2.md) | **Current** | Flexible inputs · 15-second decision strip · journey-first language · PRD-versus-code gap rule |
| [v1](v1.md) | Superseded | First version. Fixed input list, explicit `TASK` / `PARAMETERS` / `TRANSLATIONS` blocks |

## Which one to use

Use **v2**. Reach for v1 only when rerunning an audit that was originally produced with it and the two outputs have to line up.

## What it needs from you

v2 takes any subset of: a product requirement (PRD, ticket, or one sentence naming the flow), the frontend repo, the backend repo. Everything else — entry points, success condition, scope boundaries, scenario count — it derives and states in the header, so you can correct it before the second pass.

## What it returns

Scenarios covering one happy path plus the variants the code can actually produce, a ranked pain-point register scored FMEA-lite, and two change briefs. Every finding cites real identifiers: file paths, components, endpoints, handlers, tables. Anything it cannot confirm in the code is labelled `UNVERIFIED`; any frontend call with no matching handler is labelled `CONTRACT GAP`.
