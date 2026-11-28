# Chunk C12: pathway package `dm`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: C7, C11 (DONE). Next: C13 (pathway `ckd`, same shape). Decision: ED-71 (new, working choice below). ED-70 applies (stage kind `report`).

## What to build
Folder `ccis/examples/ecosystem/` (exists). One pathway `dm.pathway.json` written through the SDK by `scripts/build-ecosystem-dm-pathway.js` (under 70 lines). It imports `SPEC` and `ELEMENT_IDS` from `scripts/build-ecosystem-basic-emr.js` and `OUT`, `text` from `scripts/build-ecosystem-terms.js`, so the shared layout is the same by construction (no drift, later composition in C14 merges cleanly). Layout only: no number, rule, band or threshold.
- `manifest`: id `dm`, name "Diabetes documentation", type `pathway`, version `0.1.0`, publisher "Synthetic test publisher", standard `0.2.0`, status `draft`, `references` the eight elements as `contextual-element` with `version` `0.1.0`.
- `scope`, `intendedUse`, `exclusions`: plain text for diabetes documentation; describes information needed only; never authorises, directs or recommends treatment. Exclusions: diagnosis, severity grading and treatment choice; any product, quantity or amount of anything given; any limit on a value. Avoid the words band, target, threshold.
- `elements`: the six basic-emr elements plus `dm.core` and `dm.glycaemic-monitoring`, each `range` the bare `0.1.0`.
- `stages`: the basic-emr stage unchanged (`documentation`, kind `report`, ED-70).
- `entryForm`: the three basic-emr sections (`history`, `examination`, `assessment`) copied from `SPEC` unchanged, then a fourth section `diabetes` ("Diabetes") with element items `dm.core` and `dm.glycaemic-monitoring`, all fields, no `rowGroups`. Every element placed once.
- `contexts`: one, id `diabetes`, label "Diabetes", elements `dm.core`, `dm.glycaemic-monitoring`. The tag is known (`KNOWN_CONTEXTS`); the elements carry no field `contexts`, so nothing can disagree (ED-67).
- No `analytics`, `rules`, `nodes`, `transitions`, `timing`.
ED-71 (working choice): `dm` is a superset of `basic-emr` (all six core elements) and does not name `basic-emr` in `references`; composition dedupes by element id. Condition sections follow the shared ones.
Not in this row: `ckd`, `dm-ckd`, `htn`, `dm.foot`, `dm.dka`, element edits.

## Files to read
`scripts/build-ecosystem-basic-emr.js` (whole, 40 lines); `docs/PACKAGE-DESIGN.md` lines 79 to 86; `ccis/examples/ecosystem/dm.core.element.json` field keys only; `test/ccis-ecosystem-basic-emr.test.js` as the pattern.

## Files to change
New: `scripts/build-ecosystem-dm-pathway.js`, `dm.pathway.json`, `test/ccis-ecosystem-dm-pathway.test.js`. Earlier files untouched. Mirror row `ECO-C12` in the CCIS tracker.

## Tests (at most 8 cases)
1. `validateArtefact(pathway, { known: the eight elements })` gives no error and no row V5-036 to V5-047, V3-004 or V3-005.
2. File equals script output; SDK route equals file route (`compareRoutes`).
3. Manifest, the eight references and ranges, the one stage match the lists above.
4. Every element placed once; section ids unique; the first three sections deep-equal those in `basic-emr.pathway.json`; the fourth is `diabetes`.
5. `contexts` is exactly the one context above and every dm element is in it.
6. In memory: a context naming an element not in `elements` gives V5-044; an unknown context id gives V5-046; removing `dm.core` from `elements` gives V5-036.
7. No `analytics`, `rules`, `nodes`, `transitions`, `timing`; `intendedUse` says it does not authorise treatment.
8. LF-only, sorted keys, synthetic, no band, target or threshold word.
Run these, `ccis-ecosystem-basic-emr`, `ccis-sdk-pathway-form`, `ccis-validator-entryform`. At most 3 breakage checks (drop a placement, unknown context id, change a shared section label).

## Size guard
1 JSON, 1 script, 1 test. STOP and report if the validator refuses anything that passed in C11 (that would be a CCIS finding).

## Safety notes
Layout and references only. Wording needs the ED-2 clinician review before any use. Nothing is cleared for clinical use.

## Context impact
Reads about 5 KB; adds about 5 KB of files and 90 test lines.
