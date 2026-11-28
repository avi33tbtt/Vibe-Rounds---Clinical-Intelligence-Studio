# Chunk C8: ckd.core and ckd.kidney-function

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: C4 (DONE); C7 is the pattern. Next: C9 (htn.core). Decision: ED-68 (new, working choice below).

## What to build
Folder `ccis/examples/ecosystem/` (exists). Two elements written by ONE script through the SDK (`element()`), same file form as C7. The script `scripts/build-ecosystem-ckd.js` imports `MEASURES`, `OUT`, `text` from `scripts/build-ecosystem-terms.js` and repeats the small helpers (`f`, `concept`, E-sets, `one`, `many`) from `scripts/build-ecosystem-dm.js`. Every field carries `x-emrFieldId`.
1. `ckd.core` (both 0..1, E-A, source class clinician-entered, no concept):
   - `ckdStage` code (`ckd_stage`). Binding: code system `local` 0.1.0, value set `ecosystem.ckd-stage` 0.1.0, code `ckd-stage`, mapping mapped, note "Local code, unverified." (same shape as `dmType` in C7). The package holds six members: 1, 2, 3a, 3b, 4, 5. Nothing derives a stage from a measurement.
   - `ckdCause` string (`ckd_cause`), label "Recorded cause of kidney disease".
2. `ckd.kidney-function`, all 0..`unbounded`, E-M, source classes clinician-entered, device, imported, all OWNERS (a `concept`, no `ownedBy`):
   - `egfr` quantity (`egfr`, concept `egfr`), unit `mL/min/{1.73_m2}` (ED-68).
   - `creat` quantity `mg/dL` (`creat`, concept `creatinine`).
   - `uacr` quantity `mg/g` (`uacr`, concept `albumin-creatinine-ratio`).
   No relationship between `egfr` and `creat`: the EMR does not say it is derived and no formula is stored. A later row adds it only if a named clinician owner confirms it.
Concept objects use `local` `0.1.0` (ED-66), display copied from the package, never retyped.
Not in this row: `htn.core` (C9), `dm.foot`, `dm.dka` (C10), package edits, pathway packages.

## Files to read
`docs/ELEMENT-LIST-V0.md` lines 148 to 169 (done in this chunk); CCIS `scripts/build-ecosystem-dm.js`; `ccis/examples/ecosystem/concept-owners.json` (entries `egfr`, `creatinine`, `albumin-creatinine-ratio`); `test/ccis-ecosystem-dm.test.js` as the pattern.

## Files to change
New: `scripts/build-ecosystem-ckd.js` (under 90 lines), `ckd.core.element.json`, `ckd.kidney-function.element.json`, `test/ccis-ecosystem-ckd.test.js`. C4 to C7 files untouched. Mirror row `ECO-C8` in the CCIS tracker.

## Tests (at most 8 cases)
1. Both elements pass `validateArtefact` with no error row (a V4-011 unit warning is allowed and is named in the notes).
2. Each file equals the script output; SDK route equals file route (`compareRoutes`).
3. Field keys, types, units, cardinality, epistemic sets, sources and `x-emrFieldId` match the lists above.
4. The three concepts equal the `concept-owners.json` entries for `ckd.kidney-function` (owner element and key); no `ownedBy`; `ckd.core` has no concept.
5. `conceptOwners` over C5, C6, C7 and C8 elements reports no second owner (13 concepts, minus those not yet used by an element, as the function reports).
6. `ckdStage` binding matches the shape above; the value set exists in the package with exactly the six members.
7. No relationship member in `ckd.kidney-function`; no formula word anywhere.
8. LF-only, sorted keys, synthetic, no band, target or threshold word.
Run these, `ccis-concept-owners`, `ccis-ecosystem-terms`, `ccis-ecosystem-dm`, `ccis-sdk-elementbody`. At most 3 breakage checks (second owner, wrong unit, extra stage member).

## Size guard
2 JSON, 1 script, 1 test. STOP and report if the validator refuses the `ckdStage` binding (layer 5: `ckd-stage` is not a package concept); then split a row C4c that adds it to the package as 0.1.1.

## Safety notes
Only concept keys, units and labels. No stage cut-off, band or threshold (ED-13). The stage list and every UCUM unit need clinician confirmation (ED-2). A missing value is never zero.

## Context impact
Reads about 6 KB; adds about 6 KB of files and 80 test lines.
