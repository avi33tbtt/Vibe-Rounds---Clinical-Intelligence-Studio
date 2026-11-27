# Field map: EMR field model to CCIS field model

Prototype. Synthetic data only. Not for clinical use.

Status: draft for review in A7. Unreviewed by a clinician (ED-2 has no named owner).

## Sources read (commits as cloned on 2026-11-27)

- EMR `sets.js` lines 12 to 120 (key whitelist and validation rules), `lint.js` line 19 (type list), `emr-enhancer.html` lines 654 to 733 (built-in sets), `longitudinal-demo-content.js` lines 10 to 68 (demo sets and series), `CLINICAL_CONTEXT_MODEL.md` (observation time rules, grep only).
- CCIS `ccis/schemas/element.schema.json` and `terminology-binding.schema.json`.

Not read: the EMR conventions file (`emr-plan-chunks/01-core.md`), because this row writes no EMR code. This row cloned two repos (EMR and CCIS) because it needs both sources; the tracker allows one.

Two sets exist in two shapes. The built-in sets in `emr-enhancer.html` are client demo content. The sets in `longitudinal-demo-content.js` are the seed for the server and the explorer. They differ for `hba1c_mon`, `ckd_ctx` and add `htn_ctx`. Where they differ this doc says so, and uses the longitudinal shape for those sets, because the explorer and concepts depend on it.

## 1. Property conversion

| EMR property | CCIS field property | Conversion | What is lost or must change |
|--------------|--------------------|-----------|-----------------------------|
| `id` (letters, digits, `. _ -`, up to 64) | `key` (`^[a-z][A-Za-z0-9]*$`) | Rename to lower camel case: `bp_sys` becomes `bpSys` | Underscore, dot and hyphen are not allowed in a CCIS key. The EMR id must be kept as an `x-` extension on the field (proposed `x-emrFieldId`) so the import can find the original. A hyphenated name in `ELEMENT-LIST-V0.md` is therefore not a valid CCIS key. |
| `label` | `label` | Copy | Must not start with a space. |
| `type` `text` | `string` | Copy | None. |
| `type` `integer` | `integer` | Copy | CCIS requires `units`. A unitless integer needs the dimensionless unit `1`. |
| `type` `number` | `quantity` | Rename | CCIS requires `units`. A unitless number (`dm_duration`, `dka_ph`) needs a unit: years is `a`, pH is `[pH]`. |
| `type` `boolean` | `boolean` | Copy | None. |
| `type` `datetime` | `datetime` | Copy | CCIS requires `timeSemantics`. |
| `type` `select` with `options` | `code` | Rename | CCIS requires a `binding` for a `code` field. EMR options are plain text with no codes. Each option needs a local code in a value set. Option text is not enough. |
| `unit` (one text, up to 40) | `units` (list of UCUM strings) | Wrap in a list and convert to UCUM | Unit strings must be checked as UCUM. See section 3. |
| `repeat` | `cardinality` | `true` is min 0, max `unbounded`; absent or `false` is min 0, max 1 | EMR holds a boolean, CCIS holds integers. See section 4. |
| `required` | `cardinality.min` | `true` is min 1 | None. |
| `derived` | `sourceClasses` has `derived`; `relationships` of kind `derived-from` | Set both | The EMR does not say what a value is derived from, and no formula is stored. The source field must be named by hand. |
| `flag` | no direct property | Candidate: `constraints` (plain text) or an `x-` extension | Not decided. |
| `rationale` | no field-level property | `intent.rationale` is for the whole element | A field-level reason is lost or moves to an `x-` extension. |
| `citation` | no field-level property | `provenance.basis` is for the whole element | Same. |
| `codingPolicy` (`mode`, `systems`) | none | Dropped | CCIS has no per-field list of allowed code systems. Local vocabulary only (ED-7), so little is lost. |
| `bindings` (list of `system`, `code`, `display`, `source`, `version`) | `binding` (one object) | Take the one `local` binding | CCIS holds one binding per field. `source` (`manual`, `llm`, `sdc`) has no counterpart. |
| `linkId`, `sdcType`, `text`, `path` | none | `x-` extensions or dropped | These come from questionnaire import. |
| set `slug` | `manifest` id | Convert to `group.name` | EMR slug allows `_` and `-`. |
| set `kind` (`disease`, `care`, `template`, `custom`) | none | Not carried | The condition tag becomes `contexts` (B1). |
| set `title`, `desc` | `manifest` title, `intent.purpose` | Copy | `intent` also needs `setting`, `population`, `exclusions`, `rationale`. |
| set `citation`, `sdc` | `provenance` | Partial | `provenance` needs `origin` and `basis`. |

## 2. What the EMR cannot hold and CCIS requires

Each must be added at conversion. None has a source in the EMR definition.

| Required in CCIS | Where | Proposed source |
|------------------|-------|-----------------|
| `cardinality` | every field | From `repeat` and `required` (section 1). |
| `epistemic` (list of states) | every field | Declared by the element author. Not in the EMR set. |
| `sourceClasses` | every field | Declared by the element author. Not in the EMR set. |
| `timeSemantics` | every `date` and `datetime` field | Author. See section 5. |
| `units` | every `quantity` and `integer` | Author, from `unit` or the rules above. |
| `binding` | every `code` field | Local code per option. |
| `intent` (five parts) and `provenance` (origin, basis) | every element | Author. |

The EMR forbids numbers anywhere in a stored definition (`sets.js` walk rule). CCIS `cardinality` and `precision` hold integers. So a set materialised in the EMR from a package keeps only `repeat` and `required` as booleans. Nothing is lost for min 0 and max 1 or `unbounded`. A package field with a max of 2 or more cannot be stored as an EMR set and must be refused at import (D5).

## 3. Units

| EMR unit | Proposed UCUM | Note |
|----------|---------------|------|
| `/min` | `/min` | |
| `mmHg` | `mm[Hg]` | |
| `°C` | `Cel` | |
| `%` | `%` | |
| `mL` | `mL` | |
| `mg/dL` | `mg/dL` | Glucose and creatinine. |
| `mmol/L` | `mmol/L` | Bicarbonate, potassium. |
| `mg/g` | `mg/g` | In the demo `uacr` field. |
| `mL/min/1.73m²` | to check | The 1.73 m² normalisation needs a UCUM annotation form. The validator decides. |
| `units` (insulin) | to check | Not a clean UCUM unit. The field is excluded anyway (ED-13). |

EMR glucose and creatinine are in mg/dL. A concept has exactly one unit, so the concept keeps the EMR unit in v0 and nothing converts (ED-15).

## 4. Field by field

"Element field" uses the proposed element from `ELEMENT-LIST-V0.md`. "Fit" is one of: **same** (the proposal matches), **change** (the proposal was wrong or needs editing), **gap** (the EMR has it and the element list does not), **exclude** (see ED-13), **new** (in the list, not in the EMR).

### basic_emr (all fields `text`, single)

| EMR field | Element field | Fit |
|-----------|---------------|-----|
| `cc` | `core.presentation` chief complaint | change: single, not repeating |
| `hpi` | `core.presentation` history of present illness | same |
| `pmh` | `core.presentation` past history | same |
| `exam` | `core.examination` | change: one text field. There is no body-system group. |
| `assessment` | `core.assessment-plan` assessment | change: single, not repeating |
| `plan` | `core.assessment-plan` plan | same |

### opd_visit

| EMR field | Type | Where it goes | Fit |
|-----------|------|---------------|-----|
| `opd_visit_type` | select | no element | gap |
| `opd_cc`, `opd_hpi`, `opd_pmh` | text | `core.presentation` | same |
| `opd_onset` | text | `core.presentation` or `core.event-times` | gap |
| `opd_fsh`, `opd_ros` | text | no element | gap |
| `opd_meds` | text | section `core.medications` | same |
| `opd_allergy`, `opd_allergy_detail` | select, text | section `core.allergies` | same |
| `opd_exam_gen`, `opd_exam_sys` | text | `core.examination` | change: both single text |
| `opd_results`, `opd_ix` | text | section `core.investigations` | same |
| `opd_impression`, `opd_ddx` | text | `core.assessment-plan` | gap: no differential field |
| `opd_plan`, `opd_advice`, `opd_warn` (boolean) | text, boolean | `core.assessment-plan` | gap: advice and warning flag |
| `opd_referral`, `opd_dispo` (select), `opd_fu` | text, select | section `core.care-episode` | same |

### vitals (all `repeat`)

| EMR field | Type, unit | Element field | Fit |
|-----------|-----------|---------------|-----|
| `hr` | integer, `/min` | `core.vitals` pulse rate | same |
| `bp_sys`, `bp_dia` | integer, mmHg | systolic and diastolic pressure | same (type is integer, not number) |
| `rr` | integer, `/min` | respiratory rate | same |
| `temp` | number, °C | temperature | same |
| `spo2` | integer, % | oxygen saturation | same |
| none | | weight | **new**: no EMR source. Drop from v0 or confirm as new. |

### urine_io (single, 24-hour totals)

| EMR field | Type, unit | Element field | Fit |
|-----------|-----------|---------------|-----|
| `intake` | integer, mL, "Total intake (24h)" | fluid intake over 24 hours | change |
| `output` | integer, mL, "Total urine output (24h)" | urine output over 24 hours | change |
| `balance` | integer, mL, `derived` | net balance, derived | gap |
| `catheter` | boolean | catheter in situ | gap |
| `io_notes` | text | notes | gap |

The part 1 group of direction, fluid type, volume, start and end does not exist in the EMR. Intake and output are separate fields, so they become separate concepts. This settles open point 2 of part 4.

### glucose_ss and hba1c_mon

| EMR field | Type, unit | Element field | Fit |
|-----------|-----------|---------------|-----|
| `glu` | number, mg/dL, repeat | glucose | same |
| `ss_dose` | integer, `units` | none | exclude: a dose |
| `ss_notes` | text, "Notes / band used" | none | exclude: a band |
| `hba1c` | number, %, built-in single, longitudinal repeat | HbA1c | same (repeat in the longitudinal shape) |
| `hba1c_date` | datetime, built-in only | effective time | change: see section 5 |
| `target` | text, "Target range", built-in only | none | exclude: a target |
| `hba1c_notes` | text, longitudinal only | none | gap |
| none | | glucose timing | **new**: no EMR source. Part 3 invented it. Drop. |

### dm_ctx

| EMR field | Type | Element field | Fit |
|-----------|------|---------------|-----|
| `dm_type` | select: T1DM, T2DM, Other | diabetes type | change: option text is T1DM, T2DM, Other |
| `dm_duration` | number, "Duration (years)", no unit stored | duration | change: part 3 replaced it with a diagnosis date. See ED-14. |
| `dm_meds` | text | regimen | change: single, not repeating |
| `dm_comp` | text | complications | change: single, not repeating |
| `dm_hypo` | boolean | recent hypoglycaemia | change: boolean, not a choice. The EMR has no last-episode date. |

### ckd_ctx

| EMR field | Type, unit | Element field | Fit |
|-----------|-----------|---------------|-----|
| `ckd_stage` | select: 1, 2, 3a, 3b, 4, 5 | CKD stage | same: list comes from the EMR. Needs clinician review (ED-2). |
| `egfr` | number, mL/min/1.73m², repeat in the longitudinal shape only | eGFR | same |
| `creat` | number, mg/dL, same | creatinine | same |
| `uacr` | number, mg/g, longitudinal only | albumin-creatinine ratio | same: field is `uacr` |
| `ckd_cause` | text, "Aetiology" | cause | same |
| `renal_diet` | text, built-in only | none | exclude: a restriction. See ED-13. |

The built-in `ckd_ctx` has no `repeat` and no `uacr`. The longitudinal one has both. The split into `ckd.core` and `ckd.kidney-function` follows the longitudinal shape.

### htn_ctx (longitudinal only)

| EMR field | Type, unit | Element field | Fit |
|-----------|-----------|---------------|-----|
| `htn_type` | select: Essential, Secondary, Other | hypertension type | change: options are Essential, Secondary, Other |
| `sbp`, `dbp` | number, mmHg, repeat | systolic and diastolic pressure | same concepts as `vitals` `bp_sys` and `bp_dia` |
| `htn_notes` | text | none | gap |

This is the case the concept key exists for: `vitals.bp_sys` (integer) and `htn_ctx.sbp` (number) are two series of the same concept. Per ED-12 `core.vitals` owns it and `htn.core` references it. The two EMR types differ (integer and number), so the concept needs one type decision (see 6, point 3).

### foot_ulcer and dka_ctx

| EMR field | Type, unit | Element field | Fit |
|-----------|-----------|---------------|-----|
| `ulcer_site` | text | site | same |
| `ulcer_size` | text, "Size (cm)" | none | gap |
| `ulcer_grade` | select: Wagner 0 to 5 | none | gap |
| `ulcer_inf`, `ulcer_probe` | boolean | none | gap |
| `ulcer_abx` | text, "Antibiotics" | none | exclude: treatment |
| `dka_arr` | datetime, "ED arrival" | time of arrival | change: part 3 had "suspected time" |
| `dka_ins` | datetime, "Insulin start" | none | exclude: a treatment time. See ED-13. |
| `dka_glu` | number, mg/dL | glucose (reference) | same |
| `dka_ph` | number, no unit | venous pH | gap |
| `dka_hco3` | number, mmol/L | bicarbonate | gap |
| `dka_k` | number, mmol/L, "K⁺ before insulin" | potassium | gap |
| none | | ketone | **new**: no EMR source. Part 3 invented it. Drop. |

`ulcer_inf`, `ulcer_probe`, `ulcer_grade` and `ulcer_size` are clinical findings, not doses or bands, so they are gaps and not exclusions. A wrong "none" in the element list would hide data, so they must be added when the element list is corrected.

## 5. Time and states

- **Time.** The EMR keeps time on each observation, not on the field: `effective_at` (when it was true) and `recorded_at` (when it was entered, set by the server). CCIS holds `timeSemantics` on a date or datetime field with values `event-time`, `recorded-time`, `collection-time`. `effective_at` maps to `event-time` and `recorded_at` to `recorded-time`. `collection-time` has no EMR counterpart. EMR `datetime` fields (`hba1c_date`, `dka_arr`, `dka_ins`) are content fields that sit beside the observation time. A5 decides whether such a field duplicates the observation time.
- **Precision.** `ts` is minute precision (context model, row 15). CCIS `precision` is about decimals of a value and does not cover this.
- **States.** The EMR keeps `final` and `entered_in_error` as a status, and the demo keeps an explicit Unknown distinct from a missing value. It does not store the six CCIS epistemic states per field. The six must be declared per field by the author, and A5 must say how each maps to EMR storage.
- **24-hour totals.** `urine_io` intake and output are 24-hour totals recorded at one time. The period lives in the label, not in a field.

## 6. Concept keys in the two models

- EMR `bindings` is a list and each item has `system`, `code`. A binding `{system: "local", code: "creatinine"}` already fits ED-9. No new EMR key is needed for the definition. D3 still adds `concept_key` to observations.
- CCIS `binding` is one object: `codeSystem` `{id: "local", noVersionPublished: true}`, `code`, `mapping.status`. A `code` field needs the same single slot for its value set. A field cannot hold both a concept and a value set. Select fields do not need a concept in v0, so this is not a problem now, but it must be written down in B1.
- Concept key text (`^\S+$`) allows hyphens. Field keys do not. The concept keys in `ELEMENT-LIST-V0.md` stay valid. The field keys beside them do not.

## 7. Corrections needed in ELEMENT-LIST-V0.md

Row A4a applies these. Nothing here changes the element list yet.

1. Field names: hyphens become lower camel case keys, with the EMR id kept in `x-emrFieldId`. Types become the CCIS ones (`string`, `integer`, `quantity`, `boolean`, `code`, `date`, `datetime`). `choice` becomes `code` with a local value set. `negative` becomes `explicitly-negative`. Time meanings become `event-time` and `recorded-time`.
2. Remove fields with no EMR source: `weight`, `glucose-timing`, `ketone`, `hypoglycaemia-last-date`, the examination body-system group, `follow-up-note` as written, and the event-times and fluid-balance groups as written.
3. Rebuild `core.fluid-balance` from the five `urine_io` fields. Rebuild `dm.core`, `ckd.core`, `htn.core`, `dm.dka` from section 4.
4. Add the gaps from section 4. Choose which of them become fields and which stay out of v0.
5. Concept table: drop `weight`, `ketone` and `fluid-volume`. Add `fluid-intake-24h`, `urine-output-24h`, `venous-ph`, `bicarbonate`, `potassium`. Keep `systolic-pressure` and `diastolic-pressure` shared by `core.vitals` and `htn.core`. `glucose` is shared by `dm.glycaemic-monitoring` and `dm.dka`. Fix units from section 3. Settle the integer or number type of the blood pressure concept.
6. Update the part 4 note on the plan page: the plan says `ckd_ctx` is repeating, which is true only of the longitudinal shape.

## 8. Open points

ED-13, ED-14 and ED-15 in `DECISIONS.md` need your answer before A4a. A5 (package design) must define: the `x-emrFieldId` extension, how a `code` field's local value set is published, and the import refusal for a cardinality max above 1.
