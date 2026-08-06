# Contact Data Management

> **0. Data Foundation** &nbsp;·&nbsp; 2 activities &nbsp;·&nbsp; flows not written yet

The vocabulary used to describe a person — function, seniority, role in the buying process — and keeping each contact record true to it.

## Why these belong together

Same object: the contact record. `0.04` decides what the fields are allowed to say and `0.05` makes sure individual records actually say it. Separating them would split a rule from its enforcement.

**What still differs.** `0.04` moves to Senior Ops in the mid-term model; `0.05` stays with the SDR, with automation handling enrichment and email validation underneath.

## The activities it covers

| ID | Activity | Owner short term | Owner mid-term |
|---|---|---|---|
| 0.04 | [Refresh& Update, Identify and define contact roles and decision-maker taxonomy](0-04-refresh-update-identify-and-define-contact-roles-and-decision-maker-ta.md) | SDR | Senior Ops |
| 0.05 | [Clean and validate contact records](0-05-clean-and-validate-contact-records.md) | SDR | SDR |

`R` primary owner &nbsp;·&nbsp; `S` supporting &nbsp;·&nbsp; `·` no routine part

## Worth knowing

The matrix is explicit that this defines contact **types**. Choosing which actual person to approach is Prospecting, not Data Foundation.

---

*Grouped by the object the work acts on, not by the kind of work it is. The flows themselves are not written yet.*
