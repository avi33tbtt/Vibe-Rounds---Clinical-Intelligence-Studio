# Element list v0 (corrected)

Prototype. Synthetic data only. Not for clinical use.

Status: draft for review in A7. Rewritten in A4a from `docs/FIELD-MAP.md`. It replaces the A3a to A3d drafts, which used invented fields and invalid CCIS keys. Unreviewed by a clinician (ED-2 has no named owner). All wording is synthetic demo content.

Decisions applied: ED-13 (dose, target, band, treatment and restriction fields stay out), ED-14 (`dm_duration` kept as entered), ED-15 (EMR units kept, written as UCUM), ED-9 and ED-12 (concept keys, one owner each).

## Sources

EMR built-in sets (`emr-enhancer.html`), the longitudinal sets (`longitudinal-demo-content.js`) for `hba1c_mon`, `ckd_ctx` and `htn_ctx`, and the CCIS snake element (for the certainty pattern). Where the two EMR shapes differ the longitudinal one is used.

## Rules used in every table

- **Key** is the EMR field id converted mechanically: drop each `_` and capital the next letter (`bp_sys` becomes `bpSys`, `dm_type` becomes `dmType`). It satisfies the CCIS key rule. The EMR id is kept in `x-emrFieldId`. Labels are the EMR labels.
- **Type** is a CCIS type: `string`, `integer`, `quantity`, `boolean`, `code`, `datetime`. An EMR `select` becomes `code` with a local value set that the table lists. An EMR `number` becomes `quantity`.
- **Unit** is UCUM (see `FIELD-MAP.md` section 3). Unitless quantities carry the unit shown. `check` means the unit string must be validated before use.
- **Card** is cardinality: `0..1` single, `0..*` repeating (EMR `repeat`). No EMR field is `required`.
- **Time**: a `datetime` field has `timeSemantics` `event-time`. Observation time (`effective_at`, `recorded_at`) belongs to the observation and not to the field (FIELD-MAP section 5).
- **Epistemic** sets (CCIS names). **E-A**: absent, unknown, not-collected, not-applicable, entered-in-error. **E-M**: absent, unknown, not-collected, entered-in-error. **E-N**: E-A plus explicitly-negative. The EMR does not store these per field today, so each is the author's declaration, and A5 says how each maps.
- **Source classes**. **SC-C**: clinician-entered. **SC-M**: clinician-entered, device, imported. **SC-D**: derived.
- **Concept**: the concept key, or `no`. `ref` means the field references the owner element and does not own the concept (ED-12).
- No element holds a number, band, target or threshold except the field units listed.

## Tier 1 elements

### core.presentation

Source: set `basic_emr`. All fields single.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| cc | cc | string | | 0..1 | E-A | SC-C | no |
| hpi | hpi | string | | 0..1 | E-A | SC-C | no |
| pmh | pmh | string | | 0..1 | E-N | SC-C | no |

### core.examination

Source: set `basic_emr`.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| exam | exam | string | | 0..1 | E-N | SC-C | no |

### core.assessment-plan

Source: set `basic_emr`. `plan` records what a clinician wrote. The element never suggests, orders or automates anything.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| assessment | assessment | string | | 0..1 | E-A | SC-C | no |
| plan | plan | string | | 0..1 | E-A | SC-C | no |

### core.vitals

Source: set `vitals`. All fields repeat.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| hr | hr | integer | /min | 0..* | E-M | SC-M | pulse-rate |
| bpSys | bp_sys | integer | mm[Hg] | 0..* | E-M | SC-M | systolic-pressure |
| bpDia | bp_dia | integer | mm[Hg] | 0..* | E-M | SC-M | diastolic-pressure |
| rr | rr | integer | /min | 0..* | E-M | SC-M | respiratory-rate |
| temp | temp | quantity | Cel | 0..* | E-M | SC-M | temperature |
| spo2 | spo2 | integer | % | 0..* | E-M | SC-M | oxygen-saturation |

No weight field: the EMR set has none and A4 found no source.

### core.fluid-balance

Source: set `urine_io`. Totals over 24 hours, single. The 24-hour period is in the label, not in a field.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| intake | intake | integer | mL | 0..1 | E-M | SC-C | fluid-intake-24h |
| output | output | integer | mL | 0..1 | E-M | SC-C | urine-output-24h |
| balance | balance | integer | mL | 0..1 | E-M | SC-D | no |
| catheter | catheter | boolean | | 0..1 | E-A | SC-C | no |
| ioNotes | io_notes | string | | 0..1 | E-A | SC-C | no |

`balance` is derived (relationship `derived-from` `intake` and `output`). No formula is stored. A missing intake or output never becomes zero.

### core.event-times

Source: new. Pattern from the CCIS snake element: a datetime field with a `code` certainty field linked by a `refines` relationship. This element does not change the snake element.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| onsetTime | none | datetime (event-time) | | 0..1 | E-A | clinician-entered, patient-reported, informant-reported | no |
| onsetTimeCertainty | none | code | | 0..1 | E-A | same as `onsetTime` | no |
| arrivalTime | none | datetime (event-time) | | 0..1 | E-A | clinician-entered, imported | no |

Value set for `onsetTimeCertainty` (local, proposed): exact, approximate, unknown. This must be compared with the snake element's certainty value set before use. An unknown time is never replaced by a default.

## Section items

A `section` item is a placeholder in a pathway `entryForm`. It marks where an existing EMR area (problems, allergies and so on) appears. It is not an element. It has no fields, no concept keys, no version and no owner. The data stays in the EMR tables and the envelope. A section item holds no numbers, bands or thresholds and no logic, and its label is untrusted plain text. Analytics cannot bind to it.

| Item | Area | Envelope section | EMR tab |
|------|------|------------------|---------|
| core.problems | Problem list | `problems` | Problems |
| core.allergies | Allergy list | `allergies` | Allergies |
| core.medications | History, orders, administrations | `orders` (medication orders) and `administrations` | Medications; Orders |
| core.investigations | Orders, specimens, results, acknowledgement | `orders` (investigation orders) and `results` | Orders (results not checked) |
| core.care-episode | Admission, discharge, follow-up, referral, procedure | none | none found |

Columns filled in A5 from `envelope.js` and the EMR nav tabs; details and caveats are in `docs/PACKAGE-DESIGN.md` section 9. The `opd_visit` text fields for medicines, allergies, results, referral, disposition and follow-up stay in that EMR set and are not converted (see "Left out of v0").

## Condition elements

### dm.core

Source: set `dm_ctx` (same in both shapes). Context: diabetes. All single.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| dmType | dm_type | code | | 0..1 | E-A | SC-C | no |
| dmDuration | dm_duration | quantity | a | 0..1 | E-M | SC-C | no |
| dmMeds | dm_meds | string | | 0..1 | E-N | SC-C | no |
| dmComp | dm_comp | string | | 0..1 | E-N | SC-C | no |
| dmHypo | dm_hypo | boolean | | 0..1 | E-A | SC-C | no |

Value set for `dmType` (local, from the EMR options): T1DM, T2DM, Other. `dmDuration` is kept as entered (ED-14) and is the only quantity here without a concept key: it is not a repeating measurement. `dmMeds` records what is taken and never suggests a regimen. "Recent" in `dmHypo` is not defined here.

### dm.glycaemic-monitoring

Source: sets `glucose_ss` (field `glu`) and `hba1c_mon` (field `hba1c`, longitudinal shape). Context: diabetes. Owns the concepts glucose and hba1c.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| glu | glu | quantity | mg/dL | 0..* | E-M | SC-M | glucose |
| hba1c | hba1c | quantity | % | 0..* | E-M | SC-M | hba1c |

No target range is stored (ED-13). A missing or unknown reading is never zero.

### htn.core

Source: set `htn_ctx` (longitudinal shape only). Context: hypertension.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| htnType | htn_type | code | | 0..1 | E-A | SC-C | no |
| sbp | sbp | reference to `core.vitals.bpSys` | mm[Hg] | 0..* | E-M | SC-M | systolic-pressure (ref) |
| dbp | dbp | reference to `core.vitals.bpDia` | mm[Hg] | 0..* | E-M | SC-M | diastolic-pressure (ref) |

Value set for `htnType` (local): Essential, Secondary, Other. The keys `sbp` and `dbp` differ from `bpSys` and `bpDia`. Matching is by concept key, never by field key or label.

### ckd.core

Source: set `ckd_ctx` (the single-value part). Context: CKD. Single.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| ckdStage | ckd_stage | code | | 0..1 | E-A | SC-C | no |
| ckdCause | ckd_cause | string | | 0..1 | E-A | SC-C | no |

Value set for `ckdStage` (local, from the EMR options): 1, 2, 3a, 3b, 4, 5. A clinician must review this list (ED-2). Nothing derives a stage from a measurement.

### ckd.kidney-function

Source: set `ckd_ctx` (the repeating part, longitudinal shape). Context: CKD. Owns creatinine, eGFR and albumin-creatinine ratio.

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| egfr | egfr | quantity | mL/min/1.73m2 (check) | 0..* | E-M | SC-M | egfr |
| creat | creat | quantity | mg/dL | 0..* | E-M | SC-M | creatinine |
| uacr | uacr | quantity | mg/g | 0..* | E-M | SC-M | albumin-creatinine-ratio |

The EMR does not say whether `egfr` is derived from `creat`. No formula is stored. If a clinician owner confirms it is derived, a later row adds the relationship.

## Optional elements (unlocked only if you say so, C10)

### dm.foot

Source: set `foot_ulcer`. Context: diabetes. All single. `ulcer_abx` is left out (ED-13).

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| ulcerSite | ulcer_site | string | | 0..1 | E-N | SC-C | no |
| ulcerSize | ulcer_size | string | | 0..1 | E-N | SC-C | no |
| ulcerGrade | ulcer_grade | code | | 0..1 | E-A | SC-C | no |
| ulcerInf | ulcer_inf | boolean | | 0..1 | E-A | SC-C | no |
| ulcerProbe | ulcer_probe | boolean | | 0..1 | E-A | SC-C | no |

Value set for `ulcerGrade` (local, from the EMR options): 0, 1, 2, 3, 4, 5. A clinician must review it. `ulcerSize` is text with the unit in the label, as in the EMR.

### dm.dka

Source: set `dka_ctx`. Context: diabetes. All single. `dka_ins` is left out (ED-13).

| Key | EMR id | Type | Unit | Card | States | Src | Concept |
|-----|--------|------|------|------|--------|-----|---------|
| dkaArr | dka_arr | datetime (event-time) | | 0..1 | E-A | SC-C, imported | no |
| dkaGlu | dka_glu | reference to `dm.glycaemic-monitoring.glu` | mg/dL | 0..1 | E-M | SC-M | glucose (ref) |
| dkaPh | dka_ph | quantity | [pH] | 0..1 | E-M | SC-M | venous-ph |
| dkaHco3 | dka_hco3 | quantity | mmol/L | 0..1 | E-M | SC-M | bicarbonate |
| dkaK | dka_k | quantity | mmol/L | 0..1 | E-M | SC-M | potassium |

The label of `dkaK` says "before insulin". With `dka_ins` left out the element cannot check that, and the label is the only record of it. The element records facts and times. It does not say what to do.

## Left out of v0

Each stays in the EMR set and is not an element field.

- **ED-13:** `glucose_ss.ss_dose`, `glucose_ss.ss_notes`, `hba1c_mon.target`, `foot_ulcer.ulcer_abx`, `dka_ctx.dka_ins`, `ckd_ctx.renal_diet`.
- **Observation time, not field:** `hba1c_mon.hba1c_date` (the observation carries the time; A5 confirms).
- **Notes from longitudinal-only sets:** `hba1c_mon.hba1c_notes`, `htn_ctx.htn_notes`.
- **`opd_visit` fields with no basic_emr equivalent:** `opd_visit_type`, `opd_onset`, `opd_fsh`, `opd_ros`, `opd_meds`, `opd_allergy`, `opd_allergy_detail`, `opd_exam_gen`, `opd_exam_sys`, `opd_results`, `opd_impression`, `opd_ddx`, `opd_ix`, `opd_advice`, `opd_warn`, `opd_referral`, `opd_dispo`, `opd_fu`. A later row can add an element for the outpatient visit if you want one.
- **Meaning shared with `basic_emr` but a different EMR id:** `opd_cc`, `opd_hpi`, `opd_pmh`, `opd_plan`. They carry the meaning of `cc`, `hpi`, `pmh`, `plan`, but `x-emrFieldId` holds one id per field. A5 decides how one element maps to two EMR sets.

## Concept key table

A concept key is a binding in the `local` code system: `codeSystem` `{id: "local", noVersionPublished: true}` with the key as `code`. One owner element per concept (ED-12). One unit per concept (ED-15). "Ref" lists elements that reference the owner.

| Concept key | Meaning | Unit | Owner (element.key) | Ref | Shared |
|-------------|---------|------|---------------------|-----|--------|
| systolic-pressure | Systolic blood pressure reading | mm[Hg] | core.vitals.bpSys | htn.core.sbp | yes |
| diastolic-pressure | Diastolic blood pressure reading | mm[Hg] | core.vitals.bpDia | htn.core.dbp | yes |
| glucose | Blood glucose reading | mg/dL | dm.glycaemic-monitoring.glu | dm.dka.dkaGlu | yes |
| hba1c | HbA1c measurement | % | dm.glycaemic-monitoring.hba1c | none | no |
| creatinine | Creatinine measurement | mg/dL | ckd.kidney-function.creat | none | no |
| egfr | eGFR as recorded | mL/min/1.73m2 (check) | ckd.kidney-function.egfr | none | no |
| albumin-creatinine-ratio | Urine albumin-creatinine ratio | mg/g | ckd.kidney-function.uacr | none | no |
| pulse-rate | Pulse rate | /min | core.vitals.hr | none | no |
| respiratory-rate | Respiratory rate | /min | core.vitals.rr | none | no |
| temperature | Body temperature | Cel | core.vitals.temp | none | no |
| oxygen-saturation | Oxygen saturation | % | core.vitals.spo2 | none | no |
| fluid-intake-24h | Total fluid intake over 24 hours | mL | core.fluid-balance.intake | none | no |
| urine-output-24h | Total urine output over 24 hours | mL | core.fluid-balance.output | none | no |
| venous-ph | Venous pH | [pH] | dm.dka.dkaPh | none | no |
| bicarbonate | Bicarbonate | mmol/L | dm.dka.dkaHco3 | none | no |
| potassium | Potassium | mmol/L | dm.dka.dkaK | none | no |

16 concepts, 3 shared today, no concept with two owners. The older drafts listed `weight`, `ketone`, `fluid-volume` and `event-time`. They are gone because the EMR has no source for them.

## Open points for A5 and A7

1. **Blood pressure type.** `vitals.bp_sys` is an integer and `htn_ctx.sbp` is a number. The referencing field takes the owner's type (integer). A decimal reading from the htn set would then be refused. Confirm in A5.
2. **One element, two EMR sets.** Settled in A5: a package becomes new EMR sets, so no element maps to two sets (`PACKAGE-DESIGN.md` section 7).
3. **Local value sets.** Every `code` field needs a published local value set (`dmType`, `htnType`, `ckdStage`, `ulcerGrade`, `onsetTimeCertainty`). A5 defines where it lives. A clinician must review the stage and grade lists.
4. **Section columns** were filled in A5.
5. **Epistemic and source-class declarations** are author choices and need review. The EMR stores neither.
6. **`core.event-times`** is new and not yet in any EMR set. Check the certainty value set against the snake element.
7. **Clinician review** of every list, owner and choice is required before any use (ED-2).
