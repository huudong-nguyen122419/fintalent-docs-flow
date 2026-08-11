# Who does it & scope

> part of [0.01 · Define & maintain company type](README.md)

## Who does it

|                | SDR | KAM | Junior Ops | Senior Ops | Automation |
| -------------- | --- | --- | ---------- | ---------- | ---------- |
| **Short term** | R   | ·   | ·          | S          | ·          |
| **Mid-term**   | S   | S   | ·          | R          | S          |

`R` primary owner · `S` supporting · `·` no routine part

## Scope

|                       |                                                                                                                                                                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Flow**              | Classify every company as Sponsor / PortCo / Corporate — service firms as Advisory Firm / Law Firm / Banks / Accounting Firm / Other — and keep that classification correct through data refreshes.                                |
| **Entry points**      | Admin companies list · company detail panel · Data Management (enrichment / health)                                                                                                                                                |
| **Success condition** | The company record carries the correct type; every type-filtered list, count and downstream link reflects it.                                                                                                                      |
| **Frontend**          | `fintalent-microfrontend` · `apps/admin` (companies module + shared data layer)                                                                                                                                                    |
| **Backend**           | `fintalent-backend-microservices` · admin gateway + setting microservice (company module)                                                                                                                                          |
| **Out of scope**      | Dedupe / merge (`0.02`), contact hygiene (`0.05`), campaigns; legacy companies page noted, not traced. **Scenarios: 7.** Requirement = the feature one-liner (`0.01`) + the 2026-08-10 product ruling on the Corporate definition. |
