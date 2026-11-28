# Chunk C4: the local concept package and the owner map

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B21, A3d, B1c, B1d, B2c (all DONE). Decisions: ED-62 (new, OPEN working choice below). Next: C5, C6 (elements), then C7 to C9.

## What exists
`ccis/schemas/terminology-package.schema.json` (B1b): package with `codeSystem` `local`, `concepts` (`key`, `display`, optional `definition`, `category`, `deprecated`, `replacedBy`), optional `valueSets` (`id`, `version`, `display`, `members`), `provenance`. Validator rows CCIS-V5-017 to 021 (B1f). There is no SDK builder for a package, so the file is written from a plain table by a script (same sorted-key LF form as the B18 fixtures). `docs/ELEMENT-LIST-V0.md` lists the concepts and value sets; `docs/TERMINOLOGY-DESIGN.md` sections 2 and 3 give the key rules.

## What to build (ED-62, working choice)
Folder `ccis/examples/ecosystem/` (new), with:
1. `ecosystem.local-terms.terminology-package.json`: id `ecosystem.local-terms`, version `0.1.0`, code system `local` `0.1.0`.
   - 13 measurement concepts, category `observation` or `test`: `pulse-rate`, `systolic-pressure`, `diastolic-pressure`, `respiratory-rate`, `temperature`, `oxygen-saturation`, `fluid-intake-24h`, `urine-output-24h`, `glucose`, `hba1c`, `egfr`, `creatinine`, `albumin-creatinine-ratio`.
   - Value sets with their member concepts (category `finding`): `dm-type` (t1dm, t2dm, other), `htn-type` (essential, secondary, other), `ckd-stage` (1, 2, 3a, 3b, 4, 5), `onset-certainty` (exact, approximate, unknown). Member keys are prefixed by the set (`dm-type.t1dm`).
   - Display text plain, no number, band or threshold. `definition` only where needed to separate two concepts (for example `glucose` and `hba1c`). Provenance says: synthetic demo, ED-2 review pending.
2. `concept-owners.json`: hand-written map of each of the 13 measurement concepts to its one owner element and field key from `ELEMENT-LIST-V0.md` (for example `systolic-pressure` to `core.vitals` `bpSys`; users `htn.core` `sbp`). The element rows C6 to C9 are tested against it.
3. `scripts/build-ecosystem-terms.js` (one script, beside the other build scripts, pure table to files, no clock, no network) and `test/ccis-ecosystem-terms.test.js`.
Not in this row: `ulcerGrade` and the DKA concepts (C10 is optional; they would come as package `0.1.1`), any element file.

## Files to read
`ccis/schemas/terminology-package.schema.json` lines for `concepts` and `valueSets` (grep); `test/ccis-validator-terminology.test.js` lines 1 to 30 (a good package shape); `docs/ELEMENT-LIST-V0.md` lines 54 to 66 and 136 to 160. Look at one file in `scripts/` for the style.

## Tests (at most 8 cases)
1. The package passes `validateArtefact` with no error row (all layers).
2. The file equals the script's output (rerun gives identical bytes).
3. Concept keys are unique and match the EMR key rule; the 13 measurement keys equal the list in `concept-owners.json`.
4. Every value-set member names an existing concept; no concept is deprecated.
5. No digit appears in any display or definition except inside a key like `24h` or `3a` (allow-list in the test).
6. Each owner in `concept-owners.json` appears once; a concept used by `htn.core` is listed as a user, not a second owner (hand-checked table).
7. Files are LF-only, sorted-key, synthetic. At most 3 breakage checks (duplicate a key, drop a member concept, give a concept two owners).
Run the new test, `ccis-validator-terminology`, `ccis-terminology-package-schema`, `ccis-concept-owners`.

## Size guard
2 JSON files, 1 script (under 80 lines), 1 test. If the validator refuses the package in a way that needs a module edit, STOP and report.

## Safety notes
Concept keys and display text only. No clinical advice, number or threshold. Stage and type lists come from the EMR options and still need clinician review (ED-2).

## Context impact
Reads about 7 KB; adds about 12 KB of files and 80 test lines.
