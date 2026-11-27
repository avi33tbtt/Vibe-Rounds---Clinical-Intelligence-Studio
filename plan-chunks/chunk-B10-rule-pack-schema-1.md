# Chunk B10: rule-pack schema part 1 (manifest, scope, class, inputs, parameters)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1 (DONE), B9 (DONE). Decisions: ED-6, ED-20, ED-40 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 1, 3, 5, 7. Before B11.

## What exists
`ccis/schemas/manifest.schema.json` line 90: `artefactType` enum of six types, no `rule-pack`. `scripts/build-validators.js` line 10: the `names` map (one entry per schema). `ccis/schemas/analytics.schema.json` lines 70 to 100: the `parameters` style (closed object, `enabledByDefault` const false, `suppliedBy`). `ccis-validator.js` line 303 `TYPE_TO_SCHEMA` and `ccis-sdk.js` line 134 `ARTEFACT_TYPES` are NOT changed here (wiring is B12, SDK is B17).

## What to build (additive: every 0.1.0 artefact still validates)
1. `manifest.schema.json`: add `rule-pack` to `artefactType`. Nothing else.
2. New `ccis/schemas/rule-pack.schema.json`, closed object plus `x-`, `$id` `https://ccis.invalid/schemas/rule-pack.schema.json`. Required: `manifest` (allOf manifest ref, `type` const `rule-pack`), `class`, `scope`, `inputs`, `parameters`, `sourceNote`. `rules` and `tests` are B11 (not declared here, so a pack with them is refused until B11).
3. `class`: enum `data-quality`, `clinical-prompt` (the second is named only so the format never changes, ED-6).
4. `scope`: closed object, required `pathways` (array, `minItems` 1, each a manifest `reference` of type `pathway`).
5. `inputs`: array (`minItems` 1) of closed objects: `id` (camelCase key), `concept` (local concept code, same pattern as element-group `concept`), optional `element` (artefactId, narrows only), `read` enum `latest`, `all`, `count`. No `label`: inputs never match by label.
6. `parameters`: array (may be empty) of closed objects: `id` (key), `label` (text), `kind` enum `interval`, `count`, optional `units` (one pattern string, array `minItems` 1), `enabledByDefault` const false, `suppliedBy` (text). No `value`, `default` or `allowedRange` member: a pack carries no number.
7. `sourceNote`: one plain text (design section 7). Author, reviewer and owner stay in the manifest (checked by B13).
8. Register `ruleP` in `build-validators.js` `names` as `rulePack: 'rule-pack'`; run `npm run build` and `npm run build:app`.

ED-40 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) `rules` and `tests` wait for B11, so B10 stays small; (b) `rule-pack` is added to the manifest type enum now because the schema needs it, but `TYPE_TO_SCHEMA`, `ARTEFACT_TYPES` and the market index are left alone until B12, B17 and B15; (c) parameter kinds are `interval` and `count` only, with no allowed range, so a range cannot read as a clinical bound; (d) `class` is checked as an enum only, the version-1 ban on `clinical-prompt` is B13 and the Marketplace CI.

## Files to read
- `ccis/schemas/manifest.schema.json` lines 85 to 92.
- `ccis/schemas/analytics.schema.json` lines 70 to 100 (copy style).
- `test/ccis-group-owner-schema.test.js` lines 1 to 40 (fixture and build-equality test style).
- `scripts/build-validators.js` lines 8 to 20.

## Files to change
`manifest.schema.json` (1 line), new `rule-pack.schema.json` (about 70 lines), `scripts/build-validators.js` (1 entry), new `test/ccis-rule-pack-schema.test.js`, regenerated `ccis/validators/ccis-validators.js` and `index.html`, CCIS tracker row `ECO-B10`. Studio: `DECISIONS.md` ED-40.

## Tests (at most 8 cases, `node --test`)
1. A minimal valid pack passes `rulePack`.
2. Each required member missing is refused (manifest, class, scope, inputs, parameters, sourceNote).
3. Manifest `type` other than `rule-pack`, and `class` outside the enum, are refused.
4. Unknown member refused; `x-` member accepted; `rules` refused until B11.
5. Input with `label`, a bad `read`, or an empty `pathways`/`inputs` is refused.
6. Parameter with `value`, `default` or `allowedRange`, or `enabledByDefault` true, is refused.
7. A pre-existing element, pathway and analytics example still validates (manifest enum change is additive).
8. The generated validators equal a fresh build.
Run the new file plus `ccis-element-group-schema`, `ccis-group-owner-schema`, `ccis-demo-pathway`, `ccis-sdk-pathway`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 new schema, 1 test, 1 line in the manifest schema, 1 entry in the build script, 2 regenerated files. If any existing test needs a change, or the regenerated validators differ in more than `manifest` and the new `rulePack`, stop and split.

## Safety notes
Shape only. No number, band, threshold, default or clinical logic. Labels and notes are plain untrusted text on display.

## Context impact
Reads about 8 KB; adds about 70 JSON lines and a test of about 90 lines. Next row after this: B11 (rules, messages, tests).
