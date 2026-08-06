# Clean and validate records

> **0. Data Foundation** &nbsp;·&nbsp; 3 activities &nbsp;·&nbsp; steps not written yet

Find records that are wrong — duplicated, stale, incomplete, linked to the wrong company or the wrong owner — and fix them.

## Why these are one flow

Recurring hygiene on records that already exist, as opposed to deciding what the records should look like. Same trigger (something looks wrong), same shape (spot it, verify it, correct it), same owner today: SDR.

**What still differs.** Where each one is heading: `0.02` hands over to automation entirely in the mid-term model, `0.05` stays with the SDR with automation assisting, and `0.03` moves to Senior Ops because account ownership is a judgment call, not a lookup.

## The activities it covers

| ID | Activity | Owner short term | Owner mid-term |
|---|---|---|---|
| 0.02 | [Clean and deduplicate company records](0-02-clean-and-deduplicate-company-records.md) | SDR | Automation |
| 0.03 | [Validate company ownership and account assignment](0-03-validate-company-ownership-and-account-assignment.md) | SDR | Senior Ops |
| 0.05 | [Clean and validate contact records](0-05-clean-and-validate-contact-records.md) | SDR | SDR |

`R` primary owner &nbsp;·&nbsp; `S` supporting &nbsp;·&nbsp; `·` no routine part

---

*Grouping decided by reading the matrix, not by guessing: same act, same trigger, and in most cases the same ownership signature. The steps themselves are not written yet.*
