# Chunk C7: dm.core and dm.glycaemic-monitoring

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: C4 (DONE). Next: C8 (ckd elements). Decision: ED-67 (new, working choice below). ED-14 applies (answered: keep duration as entered).

## What to build
Folder `ccis/examples/ecosystem/` (exists). Two elements written by ONE script through the SDK (`element()`), same file form as C5 and C6. The script imports `MEASURES`, `OUT`, `text` from `scripts/build-ecosystem-terms.js` and copies the helpers `f`, `concept`, `E_A`, `E_M`, `SC_C`, `SC_M`, `one`, `many` from `scripts/build-ecosystem-core-measures.js` (export them from there only if that is a one-line change; otherwise repeat the few lines). Every field carries `x-emrFieldId`.
1. `dm.core` (all 0..1, source class clinician-entered, no concept anywhere):
   - `dmType` code (`dm_type`), E-A. Binding: code system `local` 0.1.0, value set `ecosystem.dm-type` 0.1.0, code `dm-type`, mapping mapped, note "local code, unverified" (same shape as `onsetTimeCertainty` in C6).
   - `dmDuration` quantity `a` (`dm_duration`), E-M. Kept as entered (ED-14).
   - `dmMeds` string (`dm_meds`), E-N. Records what is taken; never suggests a regimen.
   - `dmComp` string (`dm_comp`), E-N.
   - `dmHypo` boolean (`dm_hypo`), E-A. The word "recent" is not defined here; the label says "Hypoglycaemia episode reported" with no time window.
2. `dm.glycaemic-monitoring`: `glu` quantity `mg/dL` (concept `glucose`, `glu`) and `hba1c` quantity `%` (concept `hba1c`, `hba1c`). Both 0..`unbounded`, E-M, source classes clinician-entered, device, imported. Both are OWNERS: a `concept`, no `ownedBy`. No target range, no band, no formula.
E-A, E-N, E-M are the sets used in C5 and C6. Concept objects use `local` `0.1.0` (ED-66), display copied from the package, never retyped.
Not in this row: `dm.foot`, `dm.dka` (C10), `ckd.*`, `htn.core`, package edits, pathway packages.

## Files to read
`docs/ELEMENT-LIST-V0.md` lines 111 to 135 (done in this chunk); CCIS `scripts/build-ecosystem-core-measures.js` (62 lines); `ccis/examples/ecosystem/concept-owners.json` (entries `glucose`, `hba1c`); `test/ccis-ecosystem-core-measures.test.js` lines 1 to 25 as the pattern.

## Files to change
New: `scripts/build-ecosystem-dm.js` (under 90 lines), `dm.core.element.json`, `dm.glycaemic-monitoring.element.json`, `test/ccis-ecosystem-dm.test.js`. At most a one-line `export` edit in the C6 script. C4, C5, C6 files untouched. Mirror row `ECO-C7` in the CCIS tracker.

## Tests (at most 8 cases)
1. Both elements pass `validateArtefact` with no error row.
2. Each file equals the script output; SDK route equals file route (`compareRoutes`).
3. Field keys, types, units, cardinality, epistemic sets, source classes and `x-emrFieldId` match the lists above.
4. `glu` and `hba1c` concepts equal the `concept-owners.json` entries (owner element and key); no `ownedBy` on any field; each concept appears once.
5. `conceptOwners` over C5, C6 and C7 elements reports no duplicate owner.
6. `dmType` binding matches the shape above and the value set id exists in the package.
7. `dmDuration` is `quantity` with unit `a` and no concept; `dmHypo` is `boolean`.
8. LF-only, sorted keys, synthetic, no band, target or threshold word.
Run these, `ccis-concept-owners`, `ccis-ecosystem-terms`, `ccis-ecosystem-core-text`, `ccis-ecosystem-core-measures`. At most 3 breakage checks (second owner, wrong unit, wrong binding value set).

## Size guard
2 JSON, 1 script, 1 test. STOP and report if the validator refuses the `dmType` binding (layer 5: `dm-type` is not a package concept); then split a row C4b that adds it to the package as 0.1.1. Do not hack around it.

## Safety notes
Only concept keys, units and labels. No target, band or threshold (ED-13). Units are UCUM and need clinician confirmation (ED-2); `dm.core` wording also waits for the ED-2 review before any use. A missing value is never zero.

## Context impact
Reads about 6 KB; adds about 6 KB of files and 80 test lines.
