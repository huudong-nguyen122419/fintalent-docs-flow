# Who does it & scope

> part of [0.01 · Define & maintain company type](./)

## Who does it

|                | SDR | KAM | Junior Ops | Senior Ops | Automation |
| -------------- | --- | --- | ---------- | ---------- | ---------- |
| **Short term** | R   | ·   | ·          | S          | ·          |
| **Mid-term**   | S   | S   | ·          | R          | S          |

`R` primary owner · `S` supporting · `·` no routine part

## Scope

<table><thead><tr><th width="193"></th><th></th></tr></thead><tbody><tr><td><strong>Flow</strong></td><td>Classify every company as Sponsor / PortCo / Corporate — service firms as Advisory Firm / Law Firm / Banks / Accounting Firm / Other — and keep that classification correct through data refreshes.</td></tr><tr><td><strong>Entry points</strong></td><td>Admin companies list · company detail panel · Data Management (enrichment / health)</td></tr><tr><td><strong>Success condition</strong></td><td>The company record carries the correct type; every type-filtered list, count and downstream link reflects it.</td></tr><tr><td><strong>Frontend</strong></td><td><code>fintalent-microfrontend</code> · <code>apps/admin</code> (companies module + shared data layer)</td></tr><tr><td><strong>Backend</strong></td><td><code>fintalent-backend-microservices</code> · admin gateway + setting microservice (company module)</td></tr><tr><td><strong>Out of scope</strong></td><td>Dedupe / merge (<code>0.02</code>), contact hygiene (<code>0.05</code>), campaigns; legacy companies page noted, not traced. <strong>Scenarios: 7.</strong> Requirement = the feature one-liner (<code>0.01</code>) + the 2026-08-10 product ruling on the Corporate definition.</td></tr></tbody></table>
