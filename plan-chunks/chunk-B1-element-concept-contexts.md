# Chunk B1: element field `concept` and `contexts`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B0, A10 (both DONE). Decision: ED-25 (open, working choice below).

## What to build
Two optional fields on an element field (`$defs/field` in `ccis/schemas/element.schema.json`, so they apply to `fields` and to module fields):

1. `concept`: a terminology binding (`$ref` terminology-binding) narrowed to the `local` system:
   - `codeSystem` and `code` required; `codeSystem.id` is `"local"`; `mapping.status` is `"mapped"`.
   - `display` optional. No `valueSet` (value sets stay in `binding`).
   - The key format of `code` is NOT checked here; B2 checks it (A10 rule 2). The existing `binding` is not touched.
2. `contexts`: array of tags, `minItems` 1, `uniqueItems`, each tag matching `^[a-z][a-z0-9-]*$`. Which tags are known is B2, not here.

Both are optional, so every 0.1.0 element still validates. A field that carries `concept` or `contexts` is simply rejected by a validator that predates this change (additionalProperties is false); that is expected and B19 handles the version.

## Files to read (line ranges)
- `ccis/schemas/element.schema.json` lines 75 to 126 (`$defs/field`).
- `ccis/schemas/terminology-binding.schema.json` lines 1 to 63 (already read in A10).
- `test/ccis-validator-element.test.js` lines 1 to 25 (helpers: `validateArtefact`, `read`, `clone`, `ELEMENT`).
- `scripts/build-validators.js` (28 lines; no change).

## Files to change
- `ccis/schemas/element.schema.json`: one edit.
- `ccis/validators/ccis-validators.js`: regenerated with `npm run build` (generated; never hand-edited). `node scripts/build-validators.js --check` must pass.
- New `test/ccis-element-concept-schema.test.js`.
No new module. No change to `ccis-sdk-element.js` (see split).

## Tests (at most 8 cases, `node --test`)
1. Snake element with `concept` on one field validates through layer 2.
2. `concept` on a module field validates.
3. `concept` with `codeSystem.id` other than `local` is refused.
4. `concept` with `mapping.status` `unmapped` (or `lossy`) is refused.
5. `concept` without `code` is refused.
6. `contexts` valid list validates; empty list is refused; duplicate tag is refused; a tag with uppercase or a space is refused (one case, four assertions).
7. Every existing snake and ecosystem-free example element still validates (no change in diagnostics).
8. `--check` equivalent: generated validators equal a fresh build.
Run only this file, `test/ccis-validator-element.test.js`, `test/ccis-element-group-schema.test.js` and `test/schemas.test.js`. At most 3 breakage checks (for example drop the `local` const and see case 3 fail, restore).

## Size guard
1 schema edit, 1 generated file, 1 test file. If the schema edit needs more than about 25 lines or regenerating changes other validators, stop and split.

## Split off (new row B1c)
`ccis-sdk-element.js` whitelists field keys (line 28), so the SDK cannot yet write `concept` or `contexts`. Row B1c: add both to the SDK field builder with the same rules, equivalence with JSON through `compareRoutes`, one test. It comes before C4, which authors concepts through the SDK.

## Safety notes
- No numbers, bands or thresholds; `concept` carries identity only.
- `display` is plain text and untrusted on display; this row adds no UI.
- No external code system appears in `concept` (phase I uses separate maps).

## Context impact
Reads about 8 KB of source. Adds about 25 schema lines, one generated file diff, and one test of about 80 lines. Tracker, CCIS tracker (`ECO-B1`) and `DECISIONS.md` (ED-25 status) update on completion.
