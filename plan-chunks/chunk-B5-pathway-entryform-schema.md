# Chunk B5: pathway schema, `entryForm` and `contexts` (schema only)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B3 (DONE). Decisions: ED-5, ED-20, ED-33 (new, OPEN working choices). Design: `docs/PACKAGE-DESIGN.md` sections 3 and 4. Before B6.

## What exists
`ccis/schemas/pathway.schema.json` (4 KB): `additionalProperties: false` with an `x-` escape; properties `manifest`, `scope`, `intendedUse`, `exclusions`, `elements` (reference with `range`), `stages`, `nodes`, `transitions`, `timing`, `analytics`. `$defs`: `text`, `key` (camelCase), `id` (`^[a-z][a-z0-9-]*$`). The one example is `ccis/examples/snake/snake-documentation.pathway.json`.

## What to build (additive: every 0.1.0 pathway still validates)
Two optional top-level properties and a few `$defs`. Layout and grouping only: no number, no rule, no threshold.
1. `entryForm`: closed object (plus `x-`), required `sections` (array, `minItems` 1). A section: required `id` (`$defs/id`), `label` (`$defs/text`), `items` (array, `minItems` 1).
2. An item is `oneOf`:
   - kind `element`: required `kind`, `element` (manifest `artefactId`); optional `fields` (array of `$defs/key`, `minItems` 1), `rowGroups` (array of arrays of `$defs/key`, each `minItems` 1), `visibleByDefault` (boolean).
   - kind `section`: required `kind`, `section` (string, pattern `^[a-z][a-z0-9.-]*$`); optional `visibleByDefault`. No `fields`, no `rowGroups`.
3. `contexts`: array (`minItems` 1) of closed objects with `id` (`$defs/id`), `label` (`$defs/text`), `elements` (array of `artefactId`, `minItems` 1).
4. Regenerate validators (`npm run build`) and `index.html` (`npm run build:app`).

ED-33 working choices (add one `DECISIONS.md` row, OPEN, change before the build): (a) the schema checks the shape of a `section` item only; the closed list of five section ids, element membership and field existence are B6 (so the list lives in one module, not in generated code); (b) `entryForm` and `contexts` are accepted at any standard version here; version gating is B19; (c) no cross-check between `entryForm` and `elements` in the schema; (d) `contexts` `id` is a plain tag pattern here, known tags are checked in B6.

## Files to read (line ranges)
- `ccis/schemas/pathway.schema.json` whole file (4 KB).
- `ccis/schemas/element-group.schema.json` lines 112 to 135 (the `owner` object: copy the closed-object and `artefactId` ref style).
- `test/ccis-group-owner-schema.test.js` lines 1 to 40 (fixture and build-equality test style; copy).

## Files to change
`pathway.schema.json` (edit, about 70 lines of JSON); new `test/ccis-pathway-entryform-schema.test.js`; `ccis/validators/ccis-validators.js` and `index.html` regenerated; CCIS tracker row `ECO-B5`. Studio: `DECISIONS.md` `ED-33`. No validator module, no wiring change.

## Tests (at most 8 cases, `node --test`)
1. A pathway with a valid `entryForm` and `contexts` passes the pathway validator.
2. The snake example pathway (no `entryForm`) still passes unchanged.
3. An unknown property on a section, an item or a context is refused; an `x-` property is accepted.
4. Item with a wrong `kind`, or kind `element` without `element`, is refused.
5. A `section` item carrying `fields` or `rowGroups` is refused.
6. Empty `sections`, empty `items`, empty `fields`, empty `contexts` or an empty `elements` list is refused.
7. A bad section id (capital, space) or context id is refused; a number where `label` must be text is refused.
8. The generated validators equal a fresh build (as in the B3 test).
Run only the new file plus `ccis-demo-pathway`, `ccis-sdk-pathway`, `ccis-element-group-schema`, `ccis-group-owner-schema`, `ccis-market-diabetes-ckd`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 schema edit, 1 test, 2 regenerated files. If any existing test needs a change, or the regenerated validators differ in more than the pathway schema, stop and split.

## Safety notes
Layout only. No clinical logic, no number, no dose, no band. Labels are plain, untrusted text on display.

## Context impact
Reads about 8 KB; adds about 70 JSON lines and a test of about 90 lines. Next row after this: B6 (validator rules for `entryForm` and `contexts`).
