# For the tech team

> §7 surface & contract inventory + §8 gaps and open questions.
> **The only page in the space that carries file paths, handlers and endpoints.**
> Full identifiers, no code. On demand — produced when the tech team asks, or before pass 2 starts.
> Delete this block on use.

## Frontend surfaces

| Surface | Route / component | Calls | Notes |
|---|---|---|---|
| `<…>` | `<path>` | `<operation>` | `<…>` |

## Backend endpoints

| Operation | Gateway | Handler / service method | Persistence | Notes |
|---|---|---|---|---|
| `<…>` | `<path>` | `<method>` | `<collection / table>` | `<…>` |

## Contract map

Every frontend call traced to its exact handler. No handler, or no caller, is a `CONTRACT GAP`.

| FE call | BE handler | Status |
|---|---|---|
| `<…>` | `<…>` | `OK` / `CONTRACT GAP` / `field ignored` / `error swallowed` |

## Open questions & gaps

| # | Type | What is missing | Blocks | Who can answer |
|---|---|---|---|---|
| Q1 | `UNVERIFIED` | `<what is missing>` | `<pain ID / gate>` | `<role>` |
| Q2 | `CONTRACT GAP` | `<…>` | `<…>` | `<…>` |
| Q3 | `PRD GAP` | `<requirement the code does not fulfil>` | `<…>` | `<…>` |

Never invent an identifier. If it is not in the code, it is `UNVERIFIED — <what is missing>`.
