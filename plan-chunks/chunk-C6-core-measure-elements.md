# Chunk C6: vitals, fluid balance, event times

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: C4 (DONE, package and owner map). Next: C7 (dm elements). Decision: ED-66 (new, working choice below).

## What to build
Folder `ccis/examples/ecosystem/` (exists). Three elements written by ONE script through the SDK (`element()`), same file form as C4 and C5; the script imports `MEASURES`, `OUT`, `text` from `scripts/build-ecosystem-terms.js` and the E-A and E-N sets as in `scripts/build-ecosystem-core-text.js`. All fields carry `x-emrFieldId` where the EMR has an id (`none` fields get no member).
1. `core.vitals`: `hr` integer `/min` (concept `pulse-rate`), `bpSys` integer `mm[Hg]` (`systolic-pressure`), `bpDia` integer `mm[Hg]` (`diastolic-pressure`), `rr` integer `/min` (`respiratory-rate`), `temp` quantity `Cel` (`temperature`), `spo2` integer `%` (`oxygen-saturation`). Cardinality 0..`unbounded`, epistemic E-M (absent, unknown, not-collected, entered-in-error), source classes clinician-entered, device, imported. Each is the OWNER: a `concept` and no `ownedBy`.
2. `core.fluid-balance`: `intake` integer `mL` (`fluid-intake-24h`), `output` integer `mL` (`urine-output-24h`), both owners, 0..1, E-M, clinician-entered; `balance` integer `mL`, 0..1, E-M, source class `derived`, no concept; `catheter` boolean and `ioNotes` string, 0..1, E-A, clinician-entered, no concept. Two relationships `derived-from`: `balance` to `intake`, `balance` to `output`. No formula stored. Labels say "over 24 hours" (the period lives in the label, not a field).
3. `core.event-times`: `onsetTime` datetime (`event-time`; clinician-entered, patient-reported, informant-reported), `onsetTimeCertainty` code (same sources), `arrivalTime` datetime (`event-time`; clinician-entered, imported). All 0..1, E-A, no concept, no `x-emrFieldId` (source: new). `onsetTimeCertainty` binding: code system `local` 0.1.0, value set `ecosystem.onset-certainty` 0.1.0, code `onset-certainty`, mapping mapped with a note "local code, unverified". One relationship `refines`: `onsetTimeCertainty` to `onsetTime`. The element-list note to "compare with the snake certainty value set" is answered: the snake element names a different system (`ccis-demo`) and no member list, so there is nothing to merge; this element does not touch the snake element.
Concept objects: `{ codeSystem: { id: 'local', version: '0.1.0' }, code, display, mapping: { status: 'mapped' } }`, display copied from the package (import it, never retype). ED-66: use the pinned version `0.1.0` (not `noVersionPublished`, as the element list wrote) so a concept names the exact package it came from.
Not in this row: `dm.*`, `ckd.*`, `htn.core`, package edits.

## Files to read
`docs/ELEMENT-LIST-V0.md` lines 54 to 95 (done in this chunk); CCIS `ccis-sdk-element.js` lines 40 to 75; `scripts/build-ecosystem-core-text.js`; `ccis/examples/ecosystem/concept-owners.json`; `test/ccis-concept-owners.test.js` lines 1 to 25 (how `conceptOwners` is called).

## Files to change
New: `scripts/build-ecosystem-core-measures.js` (under 100 lines), 3 element JSON files, `test/ccis-ecosystem-core-measures.test.js`. No module edit; C4 and C5 files untouched. Mirror row `ECO-C6` in the CCIS tracker.

## Tests (at most 8 cases)
1. Each element passes `validateArtefact` with no error row.
2. Each file equals the script output; SDK route equals file route (`compareRoutes`).
3. Field keys, types, units, cardinality, epistemic sets and source classes match the lists above.
4. The eight owner fields' concepts equal `concept-owners.json` entries for `core.vitals` and `core.fluid-balance` (owner element and key); each concept appears once across the three files; no `ownedBy` on any field.
5. `conceptOwners` over the three elements plus C5 elements reports no duplicate owner.
6. `balance` has no concept, is `derived`, and has two `derived-from` relationships; no formula member anywhere.
7. `onsetTimeCertainty` is a `code` field with the binding above; datetime fields have `event-time`.
8. LF-only, sorted keys, synthetic, no band or threshold word.
Run these, `ccis-sdk-elementbody`, `ccis-concept-owners`, `ccis-ecosystem-terms`, `ccis-ecosystem-core-text`. At most 3 breakage checks (second owner, wrong unit, drop a relationship).

## Size guard
3 JSON, 1 script, 1 test. STOP and report if the validator refuses the `onsetTimeCertainty` binding because `code` is not a package concept (layer 5): then split a row C4a that adds concept `onset-certainty` to the package as 0.1.1 (the package changes before any publishing, so this is allowed). Do not hack around it.

## Safety notes
Only concept keys, units and labels. No target, band or threshold. Units are UCUM and still need clinician confirmation (ED-2). A missing value is never zero.

## Context impact
Reads about 7 KB; adds about 14 KB of files and 90 test lines.
