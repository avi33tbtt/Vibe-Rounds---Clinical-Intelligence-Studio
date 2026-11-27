# Chunk B12: rule-pack validator part 1 (wiring, references, operator types)

Prototype. Synthetic data only. Not for clinical use. Repo: CCIS. Needs: B11 (DONE). Decisions: ED-6, ED-40, ED-41, ED-42 (new, OPEN working choices). Design: `docs/RULE-PACK-DESIGN.md` sections 4 and 8. Before B12a and B13.

## Size check: split
The tracker row B12 covers layers 3 to 5 (references, operator types, limits, parameters, messages, instruction verbs). That is about 14 rule ids and over the 8-case test budget. Split: B12 (this chunk) is wiring, layer 3 and the layer 4 reference and operator checks. New row B12a is layer 5 (limits, concept fit, messages). B13 then needs B12a.

## What exists
`ccis-validator.js`: `TYPE_TO_SCHEMA` line 303 (no `rule-pack`), `collectRefs` line 415 (per kind), `semanticLayer` hook in `ccis-validator-semantic.js` line 197 (returns early for kinds it does not know), `RULES` list lines 45 to 70. Last rule ids: layer 4 `CCIS-V4-011`, layer 5 `CCIS-V5-047`. `ARITY` table in `ccis-validator-semantic.js` line 48 (not exported; `min`, `max`, `coalesce` have no entry).

## What to build
1. Wiring (small edits): `TYPE_TO_SCHEMA['rule-pack'] = 'rulePack'`; `collectRefs` for kind `rule-pack` lists `/scope/pathways/i` (the existing layer 3 rules then report an unknown or unsatisfied pathway); `export` the `ARITY` const; one line in `semanticLayer` calling `rulePackSemantic(doc)` for kind `rule-pack`. `ARTEFACT_TYPES` in the SDK and the market index stay as they are (B17, B15).
2. New module `ccis-validator-rulepack.js` (about 90 lines, pure) exporting `RULEPACK_RULES` and `rulePackSemantic(doc)`; add `...RULEPACK_RULES` to `RULES`:
   - `CCIS-V4-012` error: a rule reads an input id that is not declared.
   - `CCIS-V4-013` error: a rule reads a parameter id that is not declared.
   - `CCIS-V4-014` error: a test names a rule not in `rules`, or enables an undeclared parameter.
   - `CCIS-V4-015` error: a duplicate id among rules, tests, inputs or parameters.
   - `CCIS-V4-016` error: an operator has the wrong number of arguments (uses `ARITY`; operators with no entry are skipped).
   - `CCIS-V4-017` error: the root of `when` is not a true-or-false operator (`and or not is-present eq ne lt le gt ge`), or an `and`, `or`, `not` argument is a reference or literal that is not a boolean operator or a boolean literal.
   - `CCIS-V4-018` warning: a declared input or parameter that no rule reads.
3. If layers 6 to 8 report a diagnostic on a valid rule pack, guard them by kind in the chain in `ccis-validator.js` (rule-pack skipped with reason "rule-pack layers 6 to 8 are B13"). Check first; edit only if needed.
4. Rebuild `index.html` (`npm run build:app`). No schema change, so no `npm run build`.

ED-42 working choices (one `DECISIONS.md` row, OPEN, change before the build): (a) the row is split, layer 5 becomes B12a; (b) type checking is shallow (boolean-ness and arity only), because inputs carry no value type yet; (c) an input's `element` id is not resolved here (it is an id, not a versioned reference); B12a checks concepts against elements named in `manifest.references`; (d) unused declarations are a warning.

## Files to read
`ccis-validator-contexts.js` (49 lines, copy shape); `ccis-validator.js` lines 303, 375 to 380, 415 to 430, 596 to 625; `ccis-validator-semantic.js` lines 48 to 54 and 197 to 202; `test/ccis-validator-contexts.test.js` lines 1 to 40.

## Files to change
New `ccis-validator-rulepack.js`, new `test/ccis-validator-rulepack.test.js`; edits to `ccis-validator.js` (about 6 lines), `ccis-validator-semantic.js` (2 lines); rebuilt `index.html`; CCIS tracker row `ECO-B12`. Studio: ED-42, row B12a, B13 Needs.

## Tests (at most 8 cases, `node --test`)
1. A valid pack passes `validateArtefact` with no error, layers 1 to 5 passed.
2. Undeclared input and undeclared parameter in `when` (V4-012, V4-013).
3. Test with unknown rule, or enabling an undeclared parameter (V4-014).
4. Duplicate id in each of the four lists (V4-015).
5. Wrong argument count (V4-016).
6. Non-boolean root, and `and` with a reference argument (V4-017).
7. Unused input is a warning, the outcome stays `no-errors-so-far` (V4-018).
8. An unknown scope pathway gives an existing layer 3 error; the seven new ids are in `RULES`.
Run the new file plus `ccis-validator-contexts`, `ccis-validator-entryform`, `ccis-validator-semantic*`, `ccis-rule-pack-*`, `ccis-app-build-base`. At most 3 breakage checks.

## Size guard
1 module, 1 test, 3 small edits. If any existing test needs a change, or the layer 6 to 8 guard needs more than 5 lines, stop and split.

## Safety notes
Reference and shape checks only. No clinical judgement. Messages and limits are B12a; the single-value comparison ban is B13.

## Context impact
Reads about 9 KB; adds about 90 module lines and 100 test lines. Next: B12a.
