# Chunk C5: three core text elements

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: C4 (DONE). Next: C6 (vitals, fluid balance, event times).

## What to build
Folder `ccis/examples/ecosystem/` (exists from C4). Three element files written by ONE script through the SDK (`element()` from `ccis-sdk-element.js`), not by hand, in the same sorted-key LF form as C4:
- `core.presentation.element.json`: fields `cc`, `hpi` (E-A), `pmh` (E-N).
- `core.examination.element.json`: field `exam` (E-N).
- `core.assessment-plan.element.json`: fields `assessment`, `plan` (E-A). Intent says `plan` records what a clinician wrote and never suggests, orders or automates anything.
All fields: type `string`, cardinality 0..1, source class `clinician-entered`, no `concept`, no `binding`, no `contexts`, no `units`. Epistemic sets: E-A = absent, unknown, not-collected, not-applicable, entered-in-error; E-N = E-A plus explicitly-negative. Each field carries `x-emrFieldId` with the EMR id (`cc`, `hpi`, `pmh`, `exam`, `assessment`, `plan`). Manifest: id as above, version `0.1.0`, standard `0.2.0`, status `draft`, publisher "Synthetic test publisher". Provenance: synthetic demo, source set `basic_emr`, ED-2 review pending.

## Labels (open point, ED-63 proposed)
The EMR labels are not in the Studio docs and the EMR repo is not read in this row. Use plain labels: Chief complaint, History of present illness, Past medical history, Examination, Assessment, Plan. Import (D5) matches by `x-emrFieldId`, never by label, so a label difference is harmless. Say this in the summary.

## Files to read
`docs/ELEMENT-LIST-V0.md` lines 27 to 53 (done in this chunk); CCIS `ccis-sdk-element.js` lines 40 to 75 (field and element specs); `scripts/build-ecosystem-terms.js` (reuse `text` and `OUT`); `test/ccis-sdk-elementbody.test.js` lines 1 to 30 (spec shape and `verdict`).

## Files to change
New: `scripts/build-ecosystem-core-text.js` (under 80 lines, imports `text` and `OUT` from the C4 script), 3 element JSON files, `test/ccis-ecosystem-core-text.test.js`. No module edit. Does not touch the C4 files.

## Tests (at most 8 cases)
1. Each element passes `validateArtefact` with no error row, standalone.
2. Each file equals the script output byte for byte (SDK route equals file route via `compareRoutes`).
3. Field keys, types, epistemic sets and source classes match the table above.
4. No field has `concept`, `binding`, `units` or a number in a label or intent.
5. Every field has `x-emrFieldId` and the six ids are unique.
6. No concept key from `concept-owners.json` appears in these elements (no second owner).
7. LF-only, sorted keys, says synthetic.
Run the new test, `ccis-sdk-elementbody`, `ccis-ecosystem-terms`, `ccis-validator-owner`. At most 3 breakage checks (add a concept, drop `x-emrFieldId`, change an epistemic set).

## Size guard
3 JSON, 1 script, 1 test. If the SDK refuses `x-emrFieldId` on a field, STOP and report (a module edit is out of scope; propose a split row).

## Safety notes
Free text fields only. No dose, band, threshold or advice. `plan` is a record of clinician text. Clinician review still pending (ED-2).

## Context impact
Reads about 6 KB; adds about 8 KB of files and 70 test lines.
