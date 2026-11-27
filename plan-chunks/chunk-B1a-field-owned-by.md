# Chunk B1a: element field `ownedBy` (schema only)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1 (DONE 2026-11-27). Decisions: ED-12 (one owner per concept), ED-16 (user field repeats the owner's type and unit). Design: `docs/PACKAGE-DESIGN.md` section 5.

## What to build
One optional member on an element field (`$defs/field` in `ccis/schemas/element.schema.json`, so it covers `fields` and module fields):

`ownedBy`: closed object, both members required.
- `element`: the owner element id, `$ref` to the manifest `artefactId` pattern (lower-case dotted segments).
- `key`: the owner's field key, `$ref` `#/$defs/key`.

Rules in the schema:
1. `dependentRequired: { ownedBy: ["concept"] }`. A field that points at an owner must itself carry a concept. An owner field has `concept` and no `ownedBy`.
2. No version range: the owner is resolved by id against the known artefacts (highest known version, as layer 3 does at `ccis-validator.js` line 480). Version pinning is a package matter (B5, B15).
3. Optional, so every 0.1.0 element still validates. A validator that predates this change rejects the member (additionalProperties is false); B19 handles the version.

## Files to read (line ranges)
- `ccis/schemas/element.schema.json` lines 78 to 125 (`$defs/field`; the `concept` block ends near line 115).
- `ccis/schemas/manifest.schema.json` lines 84 to 90 (`artefactId`) to copy the `$ref` form.
- `test/ccis-element-concept-schema.test.js` lines 1 to 20 (helpers; copy, do not edit).
- `scripts/build-validators.js` (no change; ajv runs with `strict: true`, `dependentRequired` is draft 2020-12).

## Files to change
- `ccis/schemas/element.schema.json`: one edit, about 12 lines.
- `ccis/validators/ccis-validators.js`: regenerated with `npm run build`; `node scripts/build-validators.js --check` must pass.
- New `test/ccis-element-owned-by-schema.test.js`.
No validator module and no SDK change (see split).

## Tests (at most 8 cases, `node --test`)
1. A field with `concept` and `ownedBy` validates through layer 2.
2. `ownedBy` on a module field validates.
3. `ownedBy` without `concept` is refused.
4. `ownedBy` missing `element` or `key` is refused (two assertions).
5. `ownedBy` with an extra member is refused; an `x-` member is allowed.
6. `ownedBy.element` with uppercase or a space is refused; `key` with a leading digit is refused.
7. Every existing snake and diabetes-ckd example element still validates with no change in diagnostics.
8. Generated validators equal a fresh build.
Run only this file, `test/ccis-element-concept-schema.test.js`, `test/ccis-validator-element.test.js` and `test/schemas.test.js`. At most 3 breakage checks (drop `dependentRequired` and see case 3 fail; loosen `additionalProperties` and see case 5 fail; restore).

## Size guard
1 schema edit, 1 generated file, 1 test file. If regenerating changes any validator other than element, stop and split.

## Split off (new rows, added to the tracker)
- **B1d (SDK)**: `ccis-sdk-element.js` whitelists field keys, so the SDK cannot write `ownedBy` yet. Add it with the same rules; JSON and code routes identical through `compareRoutes`; one test. Before C4.
- **B1e (validator, layer 5)**: the cross-artefact checks the row text asked for. New pure module `ccis-validator-owner.js`, written like `ccis-validator-group.js` (resolve through `ctx.resolve`; an owner that does not resolve is skipped, never assumed): `CCIS-V5-014` error when the owner field does not exist or has no `concept`, or its concept differs from the user field's; `V5-015` error when `type` or `units` differ from the owner's (ED-16); `V5-016` warning when the user field's cardinality is wider than the owner's; self-ownership is an error. One test. It runs before B2a, which counts owners and users.

## Safety notes
- `ownedBy` carries identity only: no numbers, bands or thresholds.
- Ids and keys are plain text, untrusted on display; this row adds no UI.
- No external code system appears anywhere in this change.

## Context impact
Reads about 6 KB of source. Adds about 12 schema lines, one generated file diff and one test of about 70 lines. On completion: tracker, CCIS tracker (`ECO-B1a`), CCIS `CHANGELOG.md` only if you approve an amendment. No `DECISIONS.md` rows needed.
