# Chunk B1b: local terminology package (schema and wiring)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1, A10 (DONE). Design: `docs/TERMINOLOGY-DESIGN.md` sections 2 to 4 and 7. Decisions: ED-7, ED-9, ED-22. Before B2 and C4.

## What to build
A schema for the type `terminology-package` (the manifest enum lists it; the validator treats it as "no schema yet"). Schema only, closed objects, `x-` members allowed.

`ccis/schemas/terminology-package.schema.json`, required: `manifest`, `codeSystem`, `concepts`, `provenance`; optional `valueSets`.
- `manifest`: `allOf` the manifest schema plus `type` const `terminology-package` (copy the form used in `element-group.schema.json` lines 20 to 45).
- `codeSystem`: `{ id: "local", version: semver }`, no other system id (ED-22: external codes live in phase I maps).
- `concepts`: 1 or more of `{ key, display, definition?, category?, deprecated?, replacedBy? }`.
  - `key`: `^[a-z0-9][a-z0-9._-]{0,63}$` (the EMR rule, design rule 2).
  - `display`: plain text, 1 to 200 characters, no leading space. `definition`: up to 500.
  - `category`: enum finding, condition, test, procedure, observation, drug, other (EMR categories). Optional.
  - `deprecated`: boolean. `replacedBy`: a key; `dependentRequired: { replacedBy: ["deprecated"] }`.
- `valueSets`: 0 or more of `{ id, version (semver), display, members }`; `members` 1 or more of `{ key, display? }`, same key pattern. `id` is a lower-case dotted id.
- `provenance`: `{ origin, basis }` as other artefacts.
Resolving `replacedBy`, member keys and duplicate keys is split off (below).

Binding question from design section 4, settled as a working choice: the validator never resolves `valueSet` (grep finds no use), so keep `terminology-binding.schema.json` unchanged. A coded field keeps `code` (the concept key) and may add `valueSet`; a valueSet-only binding stays invalid. Record it as one `DECISIONS.md` row `ED-26` (OPEN, working choice, you may reverse it).

## Files to read (line ranges)
- `ccis/schemas/element-group.schema.json` lines 1 to 45 (shape, manifest `allOf`).
- `scripts/build-validators.js` line 10 (`names`), `ccis-validator.js` lines 22 to 30, 286 to 293, 365 to 372 (schema map, `TYPES_WITHOUT_SCHEMA`).
- `test/ccis-element-owned-by-schema.test.js` lines 1 to 20 (helpers; copy).

## Files to change
- New schema file (about 90 lines).
- `scripts/build-validators.js`: add `terminologyPackage: 'terminology-package'` to `names`; `npm run build` regenerates `ccis/validators/ccis-validators.js`.
- `ccis-validator.js`: import `terminologyPackage as vTerminology`, add it to the schema map, `TYPE_TO_SCHEMA['terminology-package'] = 'terminologyPackage'`, remove it from `TYPES_WITHOUT_SCHEMA`.
- New `test/ccis-terminology-package-schema.test.js`.
- `index.html`: rebuild with `npm run build:app` (it embeds the validators).
- `DECISIONS.md` (Studio): `ED-26`.
Market stays unchanged: it still refuses this type (`M-002`, `ccis-market.test.js` line 211); B15 handles that.

## Tests (at most 8 cases, `node --test`)
1. A small valid package (two concepts, one value set) passes layer 2 and the whole validator with no error.
2. Missing `codeSystem`, `concepts` or `provenance` is refused (one case, three assertions).
3. `codeSystem.id` other than `local` is refused.
4. A key with uppercase, a space, a leading dash or over 64 characters is refused.
5. `display` empty or over 200, `definition` over 500, an unknown `category` are refused.
6. `replacedBy` without `deprecated` is refused; with it, accepted.
7. A value set with no members, or a member with a bad key, is refused; an `x-` member is allowed anywhere, an unknown member is not.
8. Generated validators equal a fresh build; existing examples validate unchanged.
Run only this file, `test/ccis-validator.test.js` (two tests already fail; report, do not fix), `test/ccis-validator-element.test.js`, `test/ccis-market.test.js` and `test/ccis-app-build-base.test.js`. At most 3 breakage checks (drop `dependentRequired` and see case 6 fail; loosen the key pattern and see case 4 fail; restore).

## Size guard
1 schema, 1 generated validators file, 1 generated bundle, 2 small code edits, 1 test file. If regenerating changes an existing validator's code, or the bundle changes beyond the embedded validators, stop and split.

## Split off (new rows, added to the tracker)
- **B1f (validator layer 3 and 5, pure module)**: duplicate concept keys, `replacedBy` that names no concept or itself, value-set member keys absent from `concepts`, duplicate member keys. One module, one test. Before B2.

## Safety notes
- Vocabulary only: no numbers, bands, thresholds or external code system (SNOMED CT, LOINC).
- Display and definition are plain text, untrusted on display; no UI.

## Context impact
Reads about 8 KB of source. Adds about 90 schema lines, 2 small edits, 2 generated files and one test. On completion: Studio tracker, CCIS tracker (`ECO-B1b`), `DECISIONS.md` `ED-26`. Next row after this: B1f.
