# Chunk B2: validator rules for `concept` and `contexts`

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B1, B1f (DONE). Design: `docs/TERMINOLOGY-DESIGN.md` rule 2 and section 7; `docs/PACKAGE-DESIGN.md` sections 4 and 5. Decisions: ED-9, ED-12, ED-15, ED-27 (new, OPEN working choices). Before B2a.

## What to build
New pure module `ccis-validator-concept.js`, written like `ccis-validator-owner.js`: exports `CONCEPT_RULES`, `KNOWN_CONTEXTS` and `conceptChecks(doc)`. It checks one contextual element against itself (no `ctx.resolve`). Fields come from `fields` and module `fields`; paths are JSON pointers; messages name keys and tags only.

Rules (layer 5, ids continue after V5-021):
1. `CCIS-V5-022` error: `concept.code` fails the key format `^[a-z0-9][a-z0-9._-]{0,63}$` (design rule 2, the EMR rule).
2. `CCIS-V5-023` error: a `contexts` tag is not in `KNOWN_CONTEXTS` (PACKAGE-DESIGN section 4).
3. `CCIS-V5-024`: a field of type `quantity` or `integer` that repeats (`cardinality.max` is `unbounded` or above 1) has no `concept`. Error when `manifest.standard.version` is 0.2.0 or later; warning when it is 0.1.0, so 0.1.0 artefacts stay valid (B19).
4. `CCIS-V5-025` error: two fields of one element share a `concept.code` but differ in `type` or sorted `units`. Cross-element agreement is B1e (owner) and B2a.

ED-27 working choices (add one `DECISIONS.md` row, OPEN, you may change them before the build): `KNOWN_CONTEXTS` is `diabetes`, `ckd`, `hypertension`, the three in PACKAGE-DESIGN; a new tag is a one-line validator change with its own row. Rule 3 severity follows the standard version as above.

Wiring in `ccis-validator.js`: import, spread `CONCEPT_RULES` into `RULES`, add `...(kind === 'contextual-element' ? conceptChecks(first.doc) : [])` at the layer 5 line (about 594).

## Files to read (line ranges)
- `ccis-validator-owner.js` lines 1 to 25 (style, `fieldsOf`); do not import from it.
- `ccis-validator.js` lines 28 to 34, 50 to 58, 592 to 597.
- `test/ccis-validator-owner.test.js` lines 1 to 22 (fixture helpers; copy).
- `ccis/schemas/element.schema.json` lines 100 to 135 (`concept`, `contexts`; known).

## Files to change
- New `ccis-validator-concept.js` (about 50 lines); 3 small edits in `ccis-validator.js`.
- New `test/ccis-validator-concept.test.js`; `index.html` rebuilt with `npm run build:app`.
- `DECISIONS.md` (Studio): `ED-27`.
No schema change.

## Tests (at most 8 cases, `node --test`)
1. A field with a good concept and a known tag gives no V5-022 to 025 row.
2. A bad concept code (uppercase, space, 65 characters) gives V5-022 with the field path.
3. An unknown tag gives V5-023 at `/fields/<i>/contexts/<j>`; each known tag passes.
4. A repeating `quantity` or `integer` field with no concept: warning at standard 0.1.0, error at 0.2.0; a non-repeating or non-numeric one gives no row.
5. Two fields with the same concept code but different units, then different type, give V5-025 (two assertions); same units in another order pass.
6. A module field is checked with its module path.
7. The existing example elements still validate with no new error; list any new warning (the fixture `element-good-full.json` has one repeating numeric field without a concept).
8. An element group and a terminology package get no V5-022 to 025 row.
Run only the new file, `test/ccis-validator-owner.test.js`, `test/ccis-validator-element.test.js`, `test/ccis-element-concept-schema.test.js`, `test/ccis-app-build-base.test.js` and any test that names `element-good-full`. Old failures in `ccis-validator.test.js`: report, do not fix. At most 3 breakage checks.

## Size guard
1 new module (about 50 lines), 3 small edits, 1 test file, 1 bundle. If the module passes about 75 lines or a new warning breaks an existing test, stop and split.

## Split off (new row)
- **B2c**: a concept code used by a field but absent from the known terminology package (design section 7). It needs a rule for how an element names its package, so it gets its own chunk. Before C4.

## Safety notes
Identity and tags only: no numbers, bands or thresholds. A context never says what to do. Text in messages is plain and untrusted on display.

## Context impact
Reads about 7 KB of source. Adds about 50 source lines and one test of about 100 lines. On completion: Studio tracker, CCIS tracker (`ECO-B2`), `ED-27`. Next row after this: B2c.
