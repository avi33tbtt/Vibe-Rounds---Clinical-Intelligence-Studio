# Package design

Prototype. Synthetic data only. Not for clinical use.

Status: draft for review in A7. Unreviewed by a clinician (ED-2 has no named owner). This is a design for the CCIS 0.2.0 amendment (phase B) and the EMR import (phase D). Nothing here is implemented.

## Sources read

- CCIS `pathway.schema.json`, `manifest.schema.json`, `element.schema.json`, `terminology-binding.schema.json`.
- EMR `migrations/013_pathways.sql`, `envelope.js` (lines 160 to 185), the nav tab list in `emr-enhancer.html`, the migration file names (for table names).
- `docs/ELEMENT-LIST-V0.md` and `docs/FIELD-MAP.md` in this repo.

This row read more than five files because a design needs both ends. Not read: the CCIS validator and resolver (so how `valueSet` and a bare version are resolved is unchecked), the EMR `lint.js` binding rules and `local-terms.js`, and the EMR conventions file.

## 1. What is published

| Artefact | CCIS type | Today | Role |
|----------|-----------|-------|------|
| Element | `contextual-element` | schema exists | One thing recorded about a patient. |
| Element group | `element-group` | name in the manifest enum, no schema (B3) | Records the owner of each shared concept for a composition. |
| Pathway package | `pathway` | schema exists, needs B5 | The entry form: elements, `entryForm`, `contexts`, one stage, optional analytics and rules. |
| Local terminology | `terminology-package` | name in the manifest enum, no schema (proposed B1b) | The `local` code system, concept keys and value sets. |
| Rule pack | none | not in the enum (B10) | Designed in A6. A package only references it. |
| Analytics package | `analytics-package` | schema exists | Referenced by the pathway. |

They are published to the Marketplace repo, https://github.com/avi33tbtt/Vibe-Rounds---Marketplace, once it has its scaffold (C1 to C3). All are checksummed with `ccis-canonical-json/1` and immutable once published. A package never embeds another artefact: it references it (CCIS-REQ-014).

## 2. The pathway package

The pathway schema has `additionalProperties: false` with an `x-` escape. `entryForm`, `contexts` and `rules` can only be real properties after B5 and B10.

| Property | Required today | Content in a package |
|----------|----------------|----------------------|
| `manifest` | yes | Package id (`basic-emr`, `dm`, `ckd`, `dm-ckd`), name, `type` `pathway`, semver, publisher, standard, status. `references` may list source pathways, the element group and the local terminology. |
| `scope`, `intendedUse`, `exclusions` | yes | Plain text. States that it describes information needed and never authorises treatment. |
| `elements` | yes | References to elements. The schema accepts only `range`, not `version`. At publish the Studio writes each range as a bare version (for example `0.1.0`), which the range pattern accepts. Whether the resolver reads a bare version as exact is unchecked; B6 must test it. A caret or tilde range is refused at EMR import. |
| `stages` | yes, at least one | One stage of kind `information`, id `documentation`, with one entry and one exit line. A form package has no clinical stages. |
| `nodes`, `transitions`, `timing` | no | Empty in v0. |
| `analytics` | no | References with `required` or `optional`. |
| `entryForm` | new (B5) | Section 3. |
| `contexts` | new (B5) | Section 4. |
| `rules` | new (B10) | One reference to a rule pack. Optional. |

No number appears in a package (CCIS-REQ-025, 027). Order is array order, never a position number. Illustration, trimmed, not a file to copy:

```json
{
  "manifest": { "id": "dm", "type": "pathway", "version": "0.1.0", "status": "draft" },
  "elements": [
    { "id": "core.presentation", "type": "contextual-element", "range": "0.1.0" },
    { "id": "core.vitals", "type": "contextual-element", "range": "0.1.0" },
    { "id": "dm.core", "type": "contextual-element", "range": "0.1.0" }
  ],
  "stages": [ { "id": "documentation", "label": "Documentation", "kind": "information",
                "entry": ["A case is open"], "exit": ["The information is recorded"] } ],
  "entryForm": { "sections": [
    { "id": "history", "label": "History", "items": [
      { "kind": "element", "element": "core.presentation" },
      { "kind": "section", "section": "core.problems", "visibleByDefault": true } ] },
    { "id": "vitals", "label": "Vitals", "items": [
      { "kind": "element", "element": "core.vitals",
        "rowGroups": [["hr", "bpSys", "bpDia", "rr", "temp", "spo2"]] } ] } ] },
  "contexts": [ { "id": "diabetes", "label": "Diabetes", "elements": ["dm.core"] } ]
}
```

## 3. entryForm

Layout only. No clinical logic, no number.

- `sections`: ordered list. Each has an `id` (lowercase, hyphens), a plain-text `label` and ordered `items`.
- An item has `kind` `element` or `section`.
  - `element`: names an element that is in `elements`. Optional `fields` (ordered list of field keys; default is all fields in element order), optional `rowGroups` (lists of field keys shown on one row), optional `visibleByDefault` (boolean).
  - `section`: names one of the five section ids (`core.problems`, `core.allergies`, `core.medications`, `core.investigations`, `core.care-episode`). Optional `visibleByDefault`. No fields.
- `rowGroups` is display only. The EMR keeps each repeating field as its own series, so a row group changes no storage.
- Rules for B6: every element named exists in `elements`; every element placed at least once; each field placed once; a field named in `fields` or `rowGroups` exists in its element; section ids are from the closed list of five (an unknown one is an error); no duplicate section id.
- An element left out of the form is an error, not a hidden field.

## 4. contexts

`contexts` is a list of `{ id, label, elements }`. `id` is a lowercase tag (`diabetes`, `ckd`, `hypertension`). `elements` lists element ids from `elements`. It replaces the hard-coded `context-map.js` in the EMR (D15), which stays as a fallback.

- An element may also carry `contexts` tags (B1). When both exist they must agree. The package list wins for display; a mismatch is a validation error.
- Context tags are known to the validator (B2). An unknown tag is an error.
- A context says which elements belong to a condition. It never says what to do for the condition.

## 5. Concept ownership in an element

Fields carry a concept key as a `local` binding (ED-9). Ownership needs one more fact (new, proposed B1a): a field that uses a concept without owning it carries `ownedBy` with the owner element id and field key.

- An owner field has the concept and no `ownedBy`. A user field has the concept and `ownedBy`.
- A user field still declares its own `type`, `units`, `cardinality`, `epistemic`, `sourceClasses`, because the schema requires them. The validator checks that `type` and `units` equal the owner's (ED-16). So `htn.core.sbp` declares `integer` and `mm[Hg]`, like `core.vitals.bpSys`. The EMR demo `htn_ctx.sbp` is a number today. A set built from the package will have an integer. A decimal reading cannot be entered there.
- Cardinality of a user field may be equal to or narrower than the owner's. A wider one is a warning.
- The owner element must be present in the same package, or the package must say it is absent (a warning). A user field without its owner present is entered through its own set and its value joins the same series by concept key.
- A concept with no owner, or with two owner fields, is an error at publish (B2a, B15).

Local value sets (new, proposed B1b). A `code` field needs a `binding`. The CCIS binding names a `valueSet` by id and version, and does not hold codes. Codes and display texts must live somewhere. Proposal: one `terminology-package` artefact `local` holds the `local` code system, every concept key and every value set (`dmType`, `htnType`, `ckdStage`, `ulcerGrade`, `onsetTimeCertainty`). Elements bind to it by id and exact version. The EMR builds its select options from the display texts. A clinician must review the stage and grade lists.

## 6. Composition

Compose by reference, match by concept key, never by label (standing rule 5).

Rules:

1. Elements are deduplicated by element id. If two source packages both use `core.vitals`, the composed package lists it once.
2. Fields are matched by concept key only. Different field keys (`bpSys`, `sbp`) for one concept are normal.
3. Each concept has one owner field. If the sources already name one (`ownedBy` is present on the others), it stands. If two sources both define the concept as owner, the author chooses and the other becomes a user field (a new element version, not a silent edit).
4. A user field and its owner must agree on type and unit. A mismatch is an error.
5. The same meaning under two different concept keys is reported as a possible duplicate and never merged.
6. A shared field is shown once, at the owner's place in the form.
7. Output is deterministic: canonical JSON, stable order (sources in the order given, elements in first-seen order).
8. Composition persists as an `element-group` (members and owner per shared concept) plus a pathway whose `manifest.references` lists the group and the source pathways. No field is copied.

Conflict report (a tool output, not an artefact; it holds text and lists only):

```json
{
  "sharedElements": ["core.presentation", "core.vitals"],
  "concepts": [
    { "concept": "systolic-pressure",
      "owner": { "element": "core.vitals", "key": "bpSys" },
      "users": [ { "element": "htn.core", "key": "sbp" } ],
      "unit": "mm[Hg]",
      "status": "ok" }
  ],
  "blockers": []
}
```

`status` is one of `ok`, `needs-owner`, `second-owner`, `unit-conflict`, `type-conflict`, `possible-duplicate`. Any status other than `ok` or `possible-duplicate` is a blocker and stops the build.

Finding for C14 and E10. With the corrected element list, DM and CKD share elements (the core ones) but no concept. Only two concept pairs are shared at all: `core.vitals` with `htn.core` (blood pressure), and `dm.glycaemic-monitoring` with `dm.dka` (glucose). A `dm-ckd` conflict report would have no concepts. The plan page expected creatinine, eGFR, HbA1c and blood pressure to be shared by DM and CKD, which the real EMR sets do not support. A7 should decide whether C14 composes `dm` with `ckd` as it is, or composes something that actually has a shared concept (for example the demo `dm`, `ckd` and `htn`, or `dm` with the optional `dm.dka`). Nothing here changes C14.

## 7. Materialisation into the EMR

The EMR keeps its own tables. A package is turned into new element sets and one pathway version. The built-in sets (`vitals`, `dm_ctx` and the rest) are not touched. The artefact stays the source of truth (ED-5): its id, version and CCIS checksum are stored beside each set version and pathway version (D4).

Element to element set (ED-17):

| Package | EMR |
|---------|-----|
| element id `core.vitals` | new set slug `core-vitals` (dot becomes hyphen; the slug rule is `^[a-z0-9][a-z0-9_-]{0,63}$`); kind `custom` |
| `manifest.name`, `intent.purpose` | set `title` (200), `desc` (2000) |
| `provenance.basis` | set `citation` if at most 500 characters, else left out |
| field `x-emrFieldId`, else `key` | field `id` |
| `label` | `label` |
| `string`, `integer`, `quantity`, `boolean`, `datetime` | `text`, `integer`, `number`, `boolean`, `datetime` |
| `code` | `select`, options are the display texts of the value set in order |
| `units` (one) | `unit` (more than one is a blocker) |
| `cardinality.max` `unbounded` | `repeat: true` |
| `cardinality.max` 1 | `repeat` absent |
| `cardinality.min` 1 | `required: true` |
| `cardinality.max` above 1 | blocker: the EMR holds a boolean, not a count |
| relationship `derived-from` or source class `derived` | `derived: true` |
| `concept` | binding `{ system: "local", code: concept, source: "manual" }` |
| `epistemic`, `sourceClasses`, `timeSemantics`, `intent`, other provenance | not stored in the set; the source artefact reference keeps them |

An EMR set definition may hold no number anywhere and at most 200 fields and 100 options per field. Both are checked before staging. The EMR binding `version` must be digits as text; the `local` system has none, so it is left out.

Pathway to EMR pathway version:

- Title, description and ordered steps come from the package. A step names one exact element set version. The order is the `entryForm` order of elements. A set may appear once. At most 30 steps.
- The new `entry_form` and `contexts` columns (D4) hold the layout and contexts as declarative JSON, immutable once published. `section` items exist only there.
- The EMR computes its own pathway checksum over title, description and ordered steps. The CCIS checksum is a separate value, stored beside it. They are never compared with each other.
- Applying a pathway version pins each set version to the case (`case_applied_sets`, with `from_applied_pathway_id`). A pin never changes.
- Composed form at apply (D12): the form is built from the pinned sets. A shared field is shown once at the owner's field. Observations carry the concept key (D3), so reads by concept (D14) join the series from both sets.

## 8. Import and review path

1. **File.** The reviewer picks a Marketplace bundle file. No network (D9).
2. **Pure check** (D5, `package-import.js`). The bundle must contain the pathway, every element it references, the group if composed, and the local terminology, each with a checksum that matches. Output: proposals and a blocker report. No database write.
3. **Blockers** (the package is not stageable): range that is not an exact version; element reference not in the bundle; checksum mismatch; more than one unit; cardinality max above 1; `code` field without a value set; concept already owned by another imported element; more than 30 elements; field id or slug invalid for the EMR; a section id the EMR does not know; any text that fails the hostile-text rules (plain text only, escaped on display, no markup, no URLs followed).
4. **Stage** (D6). Append-only staging table, in the pattern of migration 033. Routes are audited and covered by the matrix tests.
5. **Accept** (D7, D8). Set drafts are created through the existing set rules and submitted for review. The person who accepts is the creator. Another user must review and publish (the existing rule that a creator does not publish alone; standing rule 8). After every set is published, the pathway draft is created with `entry_form` and `contexts`, then published through the existing pathway rules.
6. **Result.** Imported content is demo content. It carries the banner "Prototype. Synthetic data only. Not for clinical use." until a named clinician owner reviews it (ED-2).

A second import of the same artefact version with the same CCIS checksum creates nothing new. The same version with another checksum is refused (replay and tamper check).

## 9. Section items: confirmed mapping

Checked against `envelope.js` (the five sections it can send) and the EMR nav tabs. This fills the two columns that A3b and A4a left open.

| Item | Envelope section | EMR nav tab | EMR tables seen in migrations |
|------|------------------|-------------|-------------------------------|
| core.problems | `problems` | Problems | not checked |
| core.allergies | `allergies` | Allergies | not checked |
| core.medications | `orders` (medication orders) and `administrations` | Medications; Orders | `patient_medications`, `medication_orders`, `medication_reconciliations`, `medication_administrations` |
| core.investigations | `orders` (investigation orders) and `results` | Orders (whether results show there is not checked) | `investigation_orders`, `investigation_specimens`, `investigation_results`, `investigation_result_acks` |
| core.care-episode | none | none found | `admission_discharges`, `patient_followups`, `patient_referrals`, `patient_procedures` |

- The envelope section `orders` holds both medication and investigation orders, so two section items map to it.
- Medication history (`patient_medications`) has no envelope section.
- The envelope version is `0.1.0` with no sections, `0.2.0` with any section, and `0.3.0` when administrations are present.
- `core.care-episode` has no envelope section and no named tab. As an `entryForm` item it can only mark a place. The EMR import accepts it and shows no area. This is a warning, not an error.

## 10. Changes this design needs in the standard (phase B)

| Change | Row |
|--------|-----|
| `concept` and `contexts` tags on element fields | B1 (exists) |
| `ownedBy` on a field | new B1a |
| Local terminology package: code system, concepts, value sets | new B1b |
| Element group schema | B3 (exists) |
| `entryForm` and `contexts` on the pathway | B5 (exists) |
| Rule pack type and reference | B10 (exists) |
| Artefact type `rule-pack` in the manifest enum | B10 |

## 11. Open points for A6 and A7

1. **ED-16, ED-17** (OPEN in `DECISIONS.md`).
2. **Bare version as exact pin.** Confirm in B6 that the resolver reads it so.
3. **EMR `local` bindings.** EMR `lint.js` and `local-terms.js` were not read. The comments in `sets.js` mention an "unverified codes" check at submit-for-review. A `local` concept key may need a matching verified local term. This could block step 5 and must be checked before D5.
4. **C14 and E10** (section 6): the DM+CKD conflict report has no concepts.
5. **Package ids and element ids.** Whether an id may be reused across artefact types (a pathway `dm` and elements `dm.core`) was not checked.
6. **Seed data.** D18 must reconcile the demo `htn_ctx.sbp` number with the integer in the package.
7. **Clinical review** of every list, owner and value set before any use (ED-2).
