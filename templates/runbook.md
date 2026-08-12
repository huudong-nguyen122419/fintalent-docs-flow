# Runbook — template

> For a procedure someone executes: a migration, an import script, a recovery.
> Lives beside its activity: `<id>-<slug>/runbook.md`. Delete this block on use.

**Activity** `<ID>` · **Runs on** `<environment>` · **Owner** `<role>` · **Last run** `<YYYY-MM-DD>`

## What this does

`<Two sentences. Plain language — an operator has to recognise the situation.>`

## When to run it

| Trigger | Frequency | Do not run when |
|---|---|---|
| `<…>` | `<one-off / weekly / on demand>` | `<…>` |

## Before you start

* [ ] `<access, credentials, backup, dry-run flag>`

## Steps

1. `<command or action>` — expect `<observable output>`
2. `<…>`

## Verify

| Check | Expected |
|---|---|
| `<…>` | `<…>` |

## If it goes wrong

| Symptom | Cause | Recovery |
|---|---|---|
| `<…>` | `<…>` | `<…>` |

**Resumability** `<state file / idempotency notes — can this be re-run safely?>`
