# Chunk C11: pathway package `basic-emr`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: C5, C6 (DONE). Next: C12 (pathway `dm`). Decision: ED-69 (new, working choice below).

## What to build
Folder `ccis/examples/ecosystem/` (exists). One pathway `basic-emr.pathway.json` written through the SDK (`pathway()` in `ccis-sdk-pathway.js`) by `scripts/build-ecosystem-basic-emr.js` (under 80 lines); it imports `OUT`, `text` from `scripts/build-ecosystem-terms.js`. Layout only: no number, rule, band or threshold (PACKAGE-DESIGN sections 2 to 4).
- `manifest`: id `basic-emr`, name "Basic EMR documentation", type `pathway`, version `0.1.0`, publisher "Synthetic test publisher", standard `0.2.0`, status `draft`, `references` listing the six elements below as `contextual-element` with `version` `0.1.0` (as the snake pathway does). No reference to the terminology package (C15 decides how it is published).
- `scope`, `intendedUse`, `exclusions`: plain text. Describes information needed only; never authorises, directs or recommends treatment; excludes diagnosis, grading and treatment choice.
- `elements`: `core.presentation`, `core.examination`, `core.assessment-plan`, `core.vitals`, `core.fluid-balance`, `core.event-times`, each `type` `contextual-element`, `range` the bare version `0.1.0` (PACKAGE-DESIGN section 2).
- `stages`: one, id `documentation`, label "Documentation", kind `information`, entry "A case is open", exit "The information is recorded".
- `entryForm` (every element placed exactly once, all fields by default): section `history` ("History"): element `core.presentation`, element `core.event-times`, section items `core.problems`, `core.allergies`, `core.medications`. Section `examination` ("Examination"): `core.examination`; `core.vitals` with `rowGroups` `[["hr","bpSys","bpDia","rr","temp","spo2"]]`; `core.fluid-balance`. Section `assessment` ("Assessment"): `core.assessment-plan`, section item `core.investigations`.
- No `contexts`, `analytics`, `rules`, `nodes`, `transitions`, `timing`: a basic form belongs to no condition.
ED-69 (working choice): the `core.care-episode` section item is left out, because it marks a place only and raises warning V5-043; a later row may add it.
Not in this row: `dm`, `ckd` packages, condition elements, any edit to an element, package or owner map.

## Files to read
`docs/PACKAGE-DESIGN.md` lines 28 to 78 and 179 to 195 (done in this chunk); CCIS `ccis-sdk-pathway.js` lines 40 to 70; `ccis/examples/snake/snake-documentation.pathway.json` (manifest and stages shape); `test/ccis-sdk-pathway-form.test.js` lines 1 to 30 as the pattern; `scripts/build-ecosystem-dm.js` (`build` shape).

## Files to change
New: `scripts/build-ecosystem-basic-emr.js`, `basic-emr.pathway.json`, `test/ccis-ecosystem-basic-emr.test.js`. Earlier files untouched. Mirror row `ECO-C11` in the CCIS tracker.

## Tests (at most 8 cases)
1. `validateArtefact(pathway, { known: the six element files })` gives no error and no row V5-036 to V5-047 or V3-004/005.
2. File equals script output; SDK route equals file route (`compareRoutes`).
3. Manifest, the six element references and ranges, the one stage match the lists above.
4. Every element is placed once; section ids are unique; the section items are exactly the four named; no `core.care-episode`.
5. Every `rowGroups` field exists in `core.vitals`; fields are placed once.
6. In memory only: removing an element from `elements` gives V5-036; an unknown section name gives V5-041; a duplicate section id gives V5-042.
7. No `contexts`, `analytics`, `rules`; the scope text says it does not authorise treatment.
8. LF-only, sorted keys, synthetic, no band, target or threshold word, no digit-only value in the form.
Run these, `ccis-sdk-pathway-form`, `ccis-pathway-entryform-schema`, `ccis-ecosystem-core-text`, `ccis-ecosystem-core-measures`. At most 3 breakage checks (drop a placed element, wrong section name, bad row group key).

## Size guard
1 JSON, 1 script, 1 test. STOP and report if the validator refuses the bare `0.1.0` range (V3-005) or the `references` type: split a row C11a that decides the range form, and do not hack around it.

## Safety notes
Layout only; section items are places, not clinical content. Order is array order. Nothing here is cleared for clinical use (ED-2).

## Context impact
Reads about 8 KB; adds about 5 KB of files and 90 test lines.
