# Chunk C9: htn.core

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: C4 (DONE); C6 (core.vitals) and C7 are the pattern. Next: C11 (pathway `basic-emr`). No new decision.

## What to build
Folder `ccis/examples/ecosystem/` (exists). One element `htn.core` written through the SDK (`element()`) by `scripts/build-ecosystem-htn.js` (under 80 lines), same file form as C7 and C8. It imports `MEASURES`, `OUT`, `text` from `scripts/build-ecosystem-terms.js` and repeats the small helpers. The EMR set is `htn_ctx`, longitudinal shape only (ELEMENT-LIST-V0, htn.core).
1. `htnType` code (`htn_type`), 0..1, E-A, clinician-entered, no concept. Binding: `local` 0.1.0, value set `ecosystem.htn-type` 0.1.0 (members essential, secondary, other), code `htn-type`, mapping mapped, note "Local code, unverified." Label "Type of hypertension".
2. `sbp` (`sbp`) and `dbp` (`dbp`): the first USER fields. Each carries `concept` (the same object as the owner field in `core.vitals`: `systolic-pressure` and `diastolic-pressure`, display from the package) and `ownedBy` `{ element: 'core.vitals', key: 'bpSys' }` and `{ ..., key: 'bpDia' }`. Per ED-16 and validator rules V5-014 to V5-016 they repeat the owner's type `integer`, unit `mm[Hg]` and cardinality 0..`unbounded`; epistemic E-M; source classes clinician-entered, device, imported. Labels: "Systolic blood pressure (hypertension record)" and the diastolic form. Matching is by concept, never by key or label.
Not in this row: any change to `core.vitals`, package or owner map (already lists `htn.core` sbp and dbp as users), pathway packages, any blood pressure band or target.

## Files to read
`docs/ELEMENT-LIST-V0.md` lines 136 to 147 (done in this chunk); CCIS `scripts/build-ecosystem-dm.js`; `ccis-validator-owner.js` lines 7 to 60 (rules); `ccis/examples/ecosystem/core.vitals.element.json` (fields `bpSys`, `bpDia`); `concept-owners.json` (users of the two concepts).

## Files to change
New: `scripts/build-ecosystem-htn.js`, `htn.core.element.json`, `test/ccis-ecosystem-htn.test.js`. Earlier files untouched. Mirror row `ECO-C9` in the CCIS tracker.

## Tests (at most 8 cases)
1. `validateArtefact(htn.core, { known: [core.vitals] })` gives no error and no V5-014, V5-015 or V5-016 row.
2. File equals script output; SDK route equals file route (`compareRoutes`).
3. Keys, types, units, cardinality, epistemic sets, sources and `x-emrFieldId` match the lists above.
4. `sbp` and `dbp` concept objects, type, units and cardinality equal the owner fields in `core.vitals` (read from its file); `ownedBy` matches the owner map users.
5. `conceptOwners` over C5 to C9 elements: all 13 concepts `ok`, users of the two blood pressure concepts are exactly `htn.core` sbp and dbp.
6. `htnType` binding is local and its value set has exactly the three members.
7. Negative checks in memory only: a wrong owner key gives V5-014; a changed unit gives V5-015; with no `known`, the owner is skipped and no error appears.
8. LF-only, sorted keys, synthetic, no band, target or threshold word.
Run these, `ccis-concept-owners`, `ccis-ecosystem-core-measures`, `ccis-ecosystem-ckd`, `ccis-sdk-elementbody`. At most 3 breakage checks (second owner on `sbp` by dropping `ownedBy`, wrong unit, wrong owner key).

## Size guard
1 JSON, 1 script, 1 test. STOP and report if the validator refuses `ownedBy` against the C6 element for any reason other than a typo in this row: that would be a CCIS finding, not a hack.

## Safety notes
Only concept keys, units and labels. No blood pressure target, band or threshold (ED-13). Units are UCUM and need ED-2 confirmation. A missing value is never zero.

## Context impact
Reads about 6 KB; adds about 4 KB of files and 85 test lines.
