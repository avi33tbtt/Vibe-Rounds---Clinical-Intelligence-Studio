# Chunk B3: element-group schema, owner per shared concept (schema only)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1 (DONE). Decisions: ED-12 (one owner element per concept), ED-9, ED-30 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` section 6 (conflict report). Before B3a and B4.

## What exists already (CCIS row 4.5c, read in this session)
`ccis/schemas/element-group.schema.json` already has: `members` (an element reference with exact `version` or `range`, and `required` true for mandatory, false for optional), `sharedConcepts` (concept key, meaning in words, listed fields by element id and field key, `resolution` of `same-concept` or `related-not-merged`), `versionPolicy.conflict` (`refuse` or `prefer-listed-order`), `intent`, `provenance`. `ccis-validator-group.js` already gives `CCIS-V5-009` to `013`. The example is `ccis/examples/diabetes-ckd/diabetes-ckd.group.json`. So the row text "members, mandatory or optional, version constraints" is already met. Two gaps remain.

## What to build (additive, so 0.1.0 groups still validate)
1. `sharedConcepts[].owner`: optional closed object, both members required: `element` (owner element id, `$ref` to manifest `artefactId`) and `field` (`$ref` `#/$defs/key`). It names which listed field owns the concept (ED-12). Optional here; whether it is required and whether it must be one of the listed fields is B3a's validator work.
2. `sharedConcepts[].concept` accepts a local concept code. Today it must match `^[a-z][A-Za-z0-9]*$`, so `systolic-pressure` is refused although the design report uses it. Add a new `$defs/conceptKey` with pattern `^[a-z0-9][A-Za-z0-9._-]*$` (a superset of the old pattern and of the local key format) and point only `sharedConcepts[].concept` at it. The strict local key format is checked by the validator, as for elements (B2). Do not change `$defs/key` (also used by `field`). No `maxLength` (it breaks the generated code, found in B1b).

## Files to read (line ranges)
- `ccis/schemas/element-group.schema.json` lines 54 to 110 (`sharedConcepts`) and 150 to 165 (`$defs/key`).
- `ccis/schemas/manifest.schema.json` lines 84 to 90 (`artefactId`; copy the `$ref` form).
- `test/ccis-element-group-schema.test.js` lines 1 to 25 (helpers; copy, do not edit).
- `scripts/build-validators.js` (no change).

## Files to change
`ccis/schemas/element-group.schema.json` (one edit, about 20 lines); `ccis/validators/ccis-validators.js` regenerated with `npm run build` (`node scripts/build-validators.js --check` must pass); new `test/ccis-group-owner-schema.test.js`; `index.html` rebuilt (`npm ci` first if `esbuild` is missing). No validator module and no SDK change.

## Tests (at most 8 cases, `node --test`)
1. A same-concept entry with `owner` validates (`elementGroup(g) === true`).
2. `owner` missing `element`, then missing `field`, is refused (two assertions).
3. `owner` with an extra member is refused; an `x-` member is allowed.
4. `owner.element` with uppercase or a space is refused; `owner.field` with a leading digit is refused.
5. `concept` `systolic-pressure` and `bpSys` validate; `Bad Key` and `-x` are refused.
6. A group with no `owner` still validates, and `diabetes-ckd.group.json` validates with no change in diagnostics.
7. `validateArtefact` on that example still gives no new row.
8. Generated validators equal a fresh build.
Run only this file, `ccis-element-group-schema`, `ccis-diabetes-ckd-group`, `ccis-market-diabetes-ckd`, `schemas` and `ccis-app-build-base` tests. At most 3 breakage checks (drop the owner `required`, see case 2 fail; point `concept` back at `$defs/key`, see case 5 fail; restore).

## Size guard
1 schema edit, 1 generated file, 1 test, 1 bundle. If regenerating changes any validator other than `elementGroup`, stop and split.

## Split off (new row)
- **B3a**: validator rules in `ccis-validator-group.js` or a new module: the owner is one of the concept's listed fields (`V5-030`); an owner is required for `same-concept` from standard 0.2.0, a warning at 0.1.0 (`V5-031`); the concept key follows the local key format (`V5-032`). Before B4. When B4 is reached, check it against `V5-009` to `013` first: most of its text is already built.

## Safety notes
Identity only: no numbers, bands or thresholds. Ownership is never inferred from a label. Ids and keys are plain text, untrusted on display. No external code system.

## Context impact
Reads about 6 KB of source; adds about 20 schema lines, one generated file diff and one test of about 80 lines. On completion: Studio tracker, CCIS tracker (`ECO-B3`). `ED-30` added now.
