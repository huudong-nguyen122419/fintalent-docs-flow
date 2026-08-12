# For operations

> §2 journey health map + symptom check, and §4 ops playbook once it is produced.
> Plain words only — this is the page read aloud in standup. No file paths, no endpoints.
> Delete this block on use.

**`<One-line takeaway.>`**

## The journey as the user walks it

| Step | `<open>` | `<fill>` | `<assist>` | `<submit>` | `<done>` |
|---|---|---|---|---|---|
| Status | `OK` | `DEGRADED` | `SILENT-FAIL` | `OK` | `OK` |
| Pain IDs | — | `P3` | `P1`, `P5` | — | — |

`OK` — works as intended · `DEGRADED` — works, but slower or confusing · `SILENT-FAIL` — looks
like it worked and did not.

## Symptom check

Answer your decision by looking, not by memory. Support can forward this table as-is.

| What a user would report | Who hits it | Workaround today |
|---|---|---|
| `<complaint in their words>` | `<role, rough volume>` | `<what they do instead>` |

## Known errors — on demand (§4)

| Priority | User report | What is really happening | Workaround | Fixed by | Status |
|---|---|---|---|---|---|
| `P2` | `<…>` | `<…>` | `<…>` | `<pain ID>` | `<open / in progress / fixed>` |
