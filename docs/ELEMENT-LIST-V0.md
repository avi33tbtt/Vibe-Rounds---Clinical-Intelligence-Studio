# Element list v0

Prototype. Synthetic data only. Not for clinical use.

Status: parts 1 and 2 of 4 (tier 1 elements, then `section` items). Draft. Unreviewed by a clinician (ED-2 has no named owner).

## Source and limits

Element ids, captions and source EMR sets come from the plan page (`docs/ECOSYSTEM-PLAN.html`, "Standard list of contextual elements"). The fields below are proposals written for this list. They were not read from the EMR `sets.js`; row A4 compares them with the real EMR fields and records every difference. Units are shown only where a field has one. No element holds a number, band or threshold (standing rule 3). Concept keys are assigned in A3d, which names one owner element per concept (ED-12); the column here only marks fields that will need one.

## Legend

- **Repeat**: `single` is one value per element instance. `repeating` can be entered many times in one case.
- **Time**: `effective` is when the fact is true or was measured. `recorded` is when it was entered. Every field keeps both apart. Only the effective meaning is listed per field, because recorded time is always stored.
- **Epistemic states** (from the plan page): `absent`, `unknown`, `not-collected`, `not-applicable`, `negative` (explicitly negative), `entered-in-error`. Each is a state of the field, never a value. Unknown stays unknown (standing rule 6). A state set is listed per field. `value` means a normal value is also allowed.
  - **S-FULL**: value, unknown, not-collected, not-applicable, entered-in-error.
  - **S-NEG**: S-FULL plus negative (for fields where "none" is a real answer).
  - **S-MEAS**: value, unknown, not-collected, entered-in-error.
- **Concept**: `yes` means the field is expected to get a concept key in A3d. `no` means free text or layout only.

## core.presentation

Source: sets `basic_emr`, `opd_visit`. Conversion: convert.

| Field | Type | Unit | Repeat | Time | States | Concept |
|-------|------|------|--------|------|--------|---------|
| chief-complaint | text | | repeating | effective (onset as told) | S-FULL | no |
| history-of-present-illness | text | | single | effective | S-FULL | no |
| past-history | text | | single | effective | S-NEG | no |

## core.vitals

Source: set `vitals`. Conversion: convert. Repeating. Blood pressure is two concepts (systolic, diastolic), as the plan says.

| Field | Type | Unit | Repeat | Time | States | Concept |
|-------|------|------|--------|------|--------|---------|
| systolic-pressure | number | mmHg | repeating | effective (measurement) | S-MEAS | yes |
| diastolic-pressure | number | mmHg | repeating | effective (measurement) | S-MEAS | yes |
| pulse-rate | number | beats/min | repeating | effective (measurement) | S-MEAS | yes |
| respiratory-rate | number | breaths/min | repeating | effective (measurement) | S-MEAS | yes |
| temperature | number | degC | repeating | effective (measurement) | S-MEAS | yes |
| oxygen-saturation | number | % | repeating | effective (measurement) | S-MEAS | yes |
| weight | number | kg | repeating | effective (measurement) | S-MEAS | yes |

Notes: `weight` is shared by later elements and gets one owner in A3d. Whether the EMR set holds height or other fields is a question for A4.

## core.examination

Source: sets `basic_emr`, `opd_visit`. Conversion: convert.

| Field | Type | Unit | Repeat | Time | States | Concept |
|-------|------|------|--------|------|--------|---------|
| general-examination | text | | single | effective | S-FULL | no |
| system-examination | group: body-system (choice: cardiovascular, respiratory, abdominal, neurological, other) and finding (text) | | repeating | effective | S-NEG | no |

Note: the system list is a proposed layout choice, not clinical content. A4 replaces it with the EMR option list.

## core.assessment-plan

Source: sets `basic_emr`, `opd_visit`. Conversion: convert.

| Field | Type | Unit | Repeat | Time | States | Concept |
|-------|------|------|--------|------|--------|---------|
| assessment | text | | repeating | effective | S-FULL | no |
| plan | text | | single | effective | S-FULL | no |
| follow-up-note | text | | single | effective | S-NEG | no |

Note: `plan` records what a clinician wrote. The element never suggests, orders or automates anything (standing rule 4).

## core.event-times

Source: new, from the certainty pattern in the CCIS snake element. Conversion: new.

| Field | Type | Unit | Repeat | Time | States | Concept |
|-------|------|------|--------|------|--------|---------|
| event-label | choice: onset, arrival, other-event | | repeating | not applicable | S-FULL | no |
| event-time | date-time | | repeating | effective | S-FULL | yes |
| event-time-certainty | choice: exact, approximate, unknown | | repeating | effective | S-FULL | no |

Notes: one repeating group of the three fields. An `unknown` time is never replaced by a default time. The label list is a proposal. The snake element stays a separate published artefact and is not changed here.

## core.fluid-balance

Source: set `urine_io`. Conversion: convert.

| Field | Type | Unit | Repeat | Time | States | Concept |
|-------|------|------|--------|------|--------|---------|
| direction | choice: intake, output | | repeating | effective | S-FULL | no |
| fluid-type | text | | repeating | effective | S-FULL | no |
| volume | number | mL | repeating | effective (interval) | S-MEAS | yes |
| interval-start | date-time | | repeating | effective | S-MEAS | no |
| interval-end | date-time | | repeating | effective | S-MEAS | no |

Notes: the source set name suggests urine output only, while the plan says intake and output. A4 settles which fields the EMR really has. A missing volume is never zero.

## Open points for A4

1. Replace proposed fields, types, units and choice lists with the real EMR ones; list every difference.
2. Confirm effective and recorded time handling in the EMR.
3. Decide the `core.vitals` field list from the real `vitals` set.

---

# Part 2: section items

Status: draft. Unreviewed by a clinician (ED-2 has no named owner).

## What a section item is

A `section` item is a placeholder in a pathway `entryForm`. It marks where an existing EMR area (problems, allergies and so on) appears in the form. It is not an element. It has no fields of its own in the contract, no concept keys, no version and no owner. The EMR keeps storing the data in its own tables and the context envelope keeps exporting it. A pathway only says where the area sits and whether it shows by default.

Consequences:
- A section item never goes through concept ownership (ED-12). Nothing in it can be matched or composed by concept.
- Analytics cannot bind to a section item. If an analytic needs data from one of these areas, a later row must decide whether an element is needed. This part does not decide that.
- A section item holds no numbers, bands or thresholds and no clinical logic (standing rules 3 and 4).
- Text shown from a package for a section item (its label) is untrusted plain text (standing rule 9).

## Properties of a section item (proposed)

| Property | Meaning |
|----------|---------|
| id | `core.problems`, `core.allergies`, `core.medications`, `core.investigations`, `core.care-episode` |
| label | Plain text title shown on the form |
| position | Order within the `entryForm` |
| default visibility | Shown or hidden by default; the user can still open it |

These are for A5 (`docs/PACKAGE-DESIGN.md`) to confirm. Per-section field lists are deliberately absent.

## The five items

Sources are the plan page (`docs/ECOSYSTEM-PLAN.html`) and the EMR summary in it. Exact envelope section names and EMR tab names were not read from any repo. They are listed as `to confirm in A4` and must not be treated as known.

| Item | What the area stores | Matching envelope section | Matching EMR tab | Source today |
|------|----------------------|---------------------------|------------------|--------------|
| core.problems | The problem list for a case | to confirm in A4 | to confirm in A4 | EMR server tables |
| core.allergies | The allergy list for a case | to confirm in A4 | to confirm in A4 | EMR server tables |
| core.medications | Medication history, orders and administrations | to confirm in A4 (the plan page ties medications to envelope 0.3.0) | to confirm in A4 | EMR tables, envelope 0.3.0 |
| core.investigations | Orders, specimens, results and acknowledgement | to confirm in A4 (the plan page ties investigations to envelope 0.2.0) | to confirm in A4 | EMR tables, envelope 0.2.0 |
| core.care-episode | Admission, discharge, follow-up, referral and procedure | to confirm in A4 | to confirm in A4 | EMR tables and the `ipd_*` sets |

## Points for the next rows

1. A4 reads the EMR and the CCIS envelope schema by line range, then fills the two "to confirm" columns.
2. `core.care-episode` is partly stored in `ipd_*` sets today. A4 should say which parts are tables and which are sets, since a set could instead become an element.
3. A5 decides how a pathway names a section in `entryForm` and whether a missing section is an error or a warning.
4. Lab results in `core.investigations` are the likely place where analytics will want concept keys. Flag this at A3d, not here.
